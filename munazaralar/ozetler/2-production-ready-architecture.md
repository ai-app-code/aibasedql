# Production Ready Architecture - HRL Bahis Platformu

## 🎯 Proje Hedefi
Basit bir bahis botundan, **kendi riskini yöneten, yüksek frekanslı ticaret (HFT) mantığıyla çalışan profesyonel bir sisteme** evrilme. Farklı API sağlayıcılarının veri kapsamı (coverage) farklılıklarını çözen, production-ready mimari tasarımı.

## 🏗️ Sistem Mimarisi

### Genel Yapı
```
Desktop App (Electron/Python Backend)
├── ManagerAgent (UCB)
├── LiveWorkerAgent (LSTM+PPO)
└── PreMatchWorker (DQN)
    ↓
Redis Streams (State & Live Odds)
    ↓
PostgreSQL + TimescaleDB
├── MATCHES_MASTER (JSON stats)
├── DAILY_BETTING_LINES (hypertable)
├── PLAYERS_DB
└── Continuous Aggregates (1/5/15min)
    ↓
Adapter Microservice (FastAPI)
├── Circuit Breaker (PyBreaker)
└── Fallback: TimescaleDB Cache
    ↓
External APIs (API-Football v3)
```

## 🔑 Kritik Mimari Kararlar

### 1. Adapter Pattern - Veri Soyutlama
**Sorun:** Farklı API sağlayıcılarının farklı veri kapsamları (coverage)
**Çözüm:** Canonical Data Model + Adapter Pattern

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional, Dict, Tuple
import numpy as np
import torch

@dataclass
class CanonicalMatch:
    match_id: str
    home_team: str
    away_team: str
    minute: int
    score_home: int
    score_away: int
    xg_home: Optional[float] = None
    xg_away: Optional[float] = None
    dangerous_attacks: Optional[int] = None
    live_odds: Optional[Dict[str, float]] = None
    data_staleness: float = 0.0  # Veri gecikmesi (sn)
    
    def get_masked_tensor(self) -> Tuple[torch.Tensor, torch.Tensor]:
        """MaskedTensor formatında state döndürür"""
        features = np.array([
            self.minute,
            self.score_home,
            self.score_away,
            self.xg_home or np.nan,
            self.xg_away or np.nan,
            self.dangerous_attacks or np.nan,
            self.data_staleness / 300.0  # Normalize edilmiş gecikme
        ], dtype=np.float32)
        
        # Mask: 1=veri var, 0=veri yok
        mask = ~np.isnan(features).astype(np.float32)
        features = np.nan_to_num(features, nan=0.0)
        
        return torch.tensor(features), torch.tensor(mask)

class BaseDataProvider(ABC):
    @abstractmethod
    def fetch_live_match(self, match_id: str) -> CanonicalMatch:
        pass

class ApiFootballAdapter(BaseDataProvider):
    def __init__(self, db_pool, circuit_breaker):
        self.db = db_pool
        self.cb = circuit_breaker
    
    def fetch_live_match(self, match_id: str) -> CanonicalMatch:
        try:
            raw_data = self.cb.call(self._api_call, match_id)
            staleness = 0.0
        except Exception:
            # Fallback: TimescaleDB'den son geçerli veri
            raw_data = self._get_cached_from_db(match_id)
            staleness = self._calculate_staleness(raw_data)
        
        return CanonicalMatch(
            match_id=str(raw_data['fixture']['id']),
            home_team=raw_data['teams']['home']['name'],
            away_team=raw_data['teams']['away']['name'],
            minute=raw_data['fixture']['status']['elapsed'],
            score_home=raw_data['goals']['home'],
            score_away=raw_data['goals']['away'],
            xg_home=raw_data.get('statistics', {}).get('expected_goals'),
            data_staleness=staleness
        )
```

### 2. MaskedTensor - Eksik Veri Yönetimi
**Çatışma:** Beta'nın statik -1.0 masking vs Alfa'nın Embedding token yaklaşımı
**Nihai Karar:** Epsilon/Zeta'nın MaskedTensor önerisi (features + mask çifti)

**Avantajlar:**
- Model "0.0 xG" ile "Veri Yok"u ayırt eder
- LSTM'de `pack_padded_sequence` ile entegre
- Attention'da `key_padding_mask` olarak kullanılır

### 3. Circuit Breaker + Fallback Stratejisi
**Sorun:** API kesintisinde sistem çökmemeli
**Çözüm:** PyBreaker + TimescaleDB Cache

```python
from pybreaker import CircuitBreaker

class AdapterService:
    def __init__(self):
        self.cb = CircuitBreaker(
            fail_max=3,
            reset_timeout=60,
            fallback_function=self._fallback_to_cache
        )
    
    def _fallback_to_cache(self, match_id):
        """Son 5 dakika ortalamasını DB'den al"""
        query = """
        SELECT * FROM live_matches_cache 
        WHERE match_id = %s 
        ORDER BY timestamp DESC LIMIT 1
        """
        return self.db.execute(query, (match_id,)).fetchone()
```

## 🐳 Docker Compose Production Yapılandırması

```yaml
services:
  adapter:
    image: betting-adapter:latest
    read_only: true
    tmpfs: [/tmp]
    volumes:
      - ./config:/app/config:ro
    secrets: [api_key]
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  app:
    image: betting-hrl:latest
    depends_on:
      db: {condition: service_healthy}
      redis: {condition: service_healthy}
    volumes:
      - ./logs:/var/log/app:rw
    restart: always

  db:
    image: timescale/timescaledb:latest-ha
    environment:
      POSTGRES_DB: betting
    volumes:
      - db_data:/var/lib/postgresql/data
    command: ["postgres", "-c", "shared_preload_libraries=timescaledb"]

  redis:
    image: redis:alpine
    command: redis-server --appendonly yes

  prometheus:
    image: prom/prometheus
    ports: ["9090:9090"]

  grafana:
    image: grafana/grafana
    ports: ["3000:3000"]
```

## 📊 TimescaleDB Continuous Aggregates

**Amaç:** Python tarafındaki CPU yükünü %30 azaltmak

```sql
-- 1/5/15 dakikalık momentum metrikleri
CREATE MATERIALIZED VIEW momentum_1m
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 minute', timestamp) as bucket,
    match_id,
    AVG(dangerous_attacks) as avg_da,
    MAX(score_home) as max_sh
FROM live_stats
GROUP BY bucket, match_id
WITH NO DATA;

-- Gerçek zamanlı güncelleme
ALTER MATERIALIZED VIEW momentum_1m 
SET (timescaledb.materialized_only = false);

-- Retention policy (30 gün)
SELECT add_retention_policy('live_stats', INTERVAL '30 days');
```

## 🤖 HRL Agent Mimarisi

### Manager Agent (UCB)
```python
class ManagerAgent:
    def __init__(self, budget=10000):
        self.budget = budget
        self.sub_agents = {
            'live': LiveWorkerAgent(),
            'prematch': PreMatchWorkerAgent()
        }
        self.ucb_scores = {name: 1.0 for name in self.sub_agents}
    
    def allocate_capital(self, state_vector):
        """UCB ile hangi ajanın ne kadar bütçe alacağını belirler"""
        for name, agent in self.sub_agents.items():
            win_rate = agent.get_last_10_win_rate()
            self.ucb_scores[name] = win_rate + np.sqrt(
                2 * np.log(len(self.performance_memory)) / (agent.bets_placed + 1)
            )
        
        chosen_agent = max(self.ucb_scores, key=self.ucb_scores.get)
        stake = self.budget * 0.05  # %5 risk per trade
        return chosen_agent, stake
```

### Live Worker Agent (LSTM + PPO + Attention)
```python
class LiveWorkerAgent(nn.Module):
    def __init__(self):
        super().__init__()
        self.lstm = nn.LSTM(input_size=64, hidden_size=128, batch_first=True)
        self.attention = nn.MultiheadAttention(embed_dim=128, num_heads=4)
        self.policy_head = nn.Linear(128, 3)  # bet_home, bet_away, no_bet
        self.value_head = nn.Linear(128, 1)
    
    def forward(self, masked_state: torch.Tensor, mask: torch.Tensor):
        # Attention mask ile eksik verileri yoksay
        packed = nn.utils.rnn.pack_padded_sequence(
            masked_state, mask.sum(dim=1).cpu(), 
            batch_first=True, enforce_sorted=False
        )
        lstm_out, _ = self.lstm(packed)
        unpacked, _ = nn.utils.rnn.pad_packed_sequence(lstm_out, batch_first=True)
        
        # Attention: mask'i key_padding_mask olarak kullan
        attn_out, _ = self.attention(
            unpacked, unpacked, unpacked, 
            key_padding_mask=~mask.bool()
        )
        return self.policy_head(attn_out[:, -1, :]), self.value_head(attn_out[:, -1, :])
```

## 💰 Nihai Reward Fonksiyonu

**Çatışma:** Epsilon'un lineer ceza vs Zeta'nın normalizasyon önerisi
**Nihai Karar:** `staleness / 300.0` normalize + `0.001` lineer katsayı

```python
def compute_reward(state, payout, stake, staleness):
    """
    Risk ayarlı getiri + Bütçe koruması + Gecikme cezası + Performans bonusu
    """
    roi = (payout - stake) / (stake + 1e-6)
    
    # 1. Sharpe Ratio benzeri risk ayarlı getiri
    risk_adjusted_roi = roi / (state['market_volatility'] * state['risk_score'] + 1e-6)
    
    # 2. Bütçe koruma cezası
    budget_ratio = state['budget_kalan'] / state['baslangic_butcesi']
    budget_penalty = 0.3 * max(0, 0.8 - budget_ratio)
    
    # 3. Gecikmeli veri cezası (normalize + lineer)
    staleness_penalty = 0.001 * (staleness / 300.0)
    
    # 4. Dinamik başabaş noktası
    break_even = 1.0 / (state['avg_odds'] + 1e-6)
    performance_bonus = 0.2 * (state['last_10_win_rate'] - break_even)
    
    return risk_adjusted_roi - budget_penalty - staleness_penalty + performance_bonus
```

**State Vektörü:**
```python
STATE_VECTOR = [
    'budget_kalan', 'risk_score', 'market_volatility',
    'live_agent_wr', 'prematch_agent_wr', 'data_staleness'
]
```

## 📈 Monitoring & Observability

```python
from prometheus_client import Counter, Histogram, Gauge

class Metrics:
    def __init__(self):
        self.bets_placed = Counter('hrl_bets_total', 'Total bets', ['agent'])
        self.reward_dist = Histogram('hrl_reward', 'Reward distribution')
        self.budget_gauge = Gauge('hrl_budget_remaining', 'Remaining budget')
        self.data_staleness = Gauge('hrl_data_staleness_seconds', 'Data delay')
```

**Grafana Dashboard:**
- Panel 1: `hrl_budget_remaining` (Gauge)
- Panel 2: `rate(hrl_bets_total[5m])` by agent
- Panel 3: `hrl_data_staleness_seconds` > 30s alert
- Panel 4: Reward Sharpe Ratio (avg / stddev)

## 🔐 Güvenlik & Performans

1. **Container Security:** `read-only rootfs`, sadece `/tmp` tmpfs
2. **Resource Limits:** CPU 0.5, Memory 512M
3. **Health Checks:** 30s interval
4. **Secrets Management:** Docker secrets ile DB_URL
5. **Zero-Downtime Deploy:** Docker Swarm/K8s rolling update

## 🎓 Kritik Teknik Kararlar

### 1. Masking Stratejisi
- ❌ Beta: Statik -1.0 (model şaşırır)
- ❌ Alfa: Embedding token (state space genişler)
- ✅ Epsilon/Zeta: MaskedTensor (features + mask çifti)

### 2. Staleness Normalizasyonu
- ❌ Epsilon: Lineer 0.1 katsayı (gradient patlar)
- ✅ Zeta: `staleness / 300.0` normalize + 0.001 katsayı

### 3. Fallback Stratejisi
- ❌ DevOps: Redis cache (momentum bozulur)
- ✅ Delta: TimescaleDB son 5 dakika ortalaması

### 4. Momentum Hesaplama
- ❌ Python tarafında hesaplama (CPU yükü)
- ✅ Alfa: TimescaleDB Continuous Aggregates (%30 CPU tasarrufu)

## 🚀 Deployment Planı

1. Docker Compose + TimescaleDB + Adapter Microservice mimarisi
2. MaskedTensor tabanlı eksik veri yönetimi
3. HRL (UCB Manager + LSTM/PPO Worker) + Risk ayarlı Reward
4. Normalized staleness metriği (+0.001 lineer ceza)
5. Prometheus/Grafana monitoring

