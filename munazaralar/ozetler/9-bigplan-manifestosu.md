# BIGPLAN Manifestosu - AI Tabanlı Futbol Analiz Sistemi

## 📋 Genel Bilgiler
- **Oluşturma Tarihi:** 01.01.2026
- **Kaynak:** 3 münazara transcript dosyası
- **Amaç:** Tüm münazara kararlarını tek manifestoda birleştirme
- **Tur Sayısı:** 5 tur (4 karar turu + 1 onay)
- **Katılımcılar:** 10 AI uzman (Nexus, DevOps, Alfa, Beta, Gamma, Delta, Epsilon, Zeta, Theta, Kappa)

---

# 🎯 BÖLÜM 1: SİSTEM VİZYONU

## 1.1 Temel Amaç
API ile futbol verileri çekerek ML/DQL/RL/RDQL algoritmalarıyla canlı ve tarihsel veriler üzerinde tahmin sistemi geliştirmek.

## 1.2 Sistem Felsefesi
Sistem bir **"işletim sistemi"** gibi tasarlanmıştır:
- **Kernel:** Event Bus + State Machine
- **Plants (Tesisler):** Modüler, pluggable bileşenler
- **Contracts:** Her tesisin uyması gereken interface
- **Yeni özellik = Yeni tesis** (mevcut kod değişmez)

## 1.3 V1 Odak Tesisleri
1. **DataPlant:** Veri toplama
2. **IntelligencePlant:** Tahmin üretme
3. **BootstrapPlant:** Cold Start yönetimi

---

# 📌 BÖLÜM 2: VERİ KATMANI

## 2.1 API Seçimi
**Karar:** API-Football v3
- 800+ lig desteği
- Canlı + tarihsel veri
- xG, şut, korner, tehlikeli atak detayları

## 2.2 Canonical Data Model
```python
@dataclass
class CanonicalMatch:
    match_id: str
    home_team: str
    away_team: str
    minute: int
    score_home: int
    score_away: int
    status: str
    # Coverage-dependent (Optional)
    xg_home: Optional[float] = None
    xg_away: Optional[float] = None
    possession_home: Optional[float] = None
    dangerous_attacks_home: Optional[int] = None
    live_odds: Optional[Dict[str, float]] = None
    coverage_mask: Dict[str, bool]  # EKSİK 1 çözümü
```

## 2.3 Twin Database Mimarisi

### Ana Veri Depolama Sistemleri

| Veritabanı | Rol | Özellikler |
|-----------|-----|------------|
| **ClickHouse** | Ana canlı veri | 1M/s ingestion, ReplacingMergeTree idempotent upsert |
| **TimescaleDB** | OLTP işlemler | Hypertable, continuous aggregates |
| **Neo4j** | Knowledge Graph | Takım formasyonları, sakatlıklar, oyuncu ilişkileri |
| **Milvus** | Vector Store | 128-boyutlu embedding storage, gRPC hızlı erişim |
| **Redis** | Cache + Online Store | TTL 30s, Lua CAS versioning |
| **Delta Lake + Hudi** | Offline Store | Merge-on-Read, %60 write amplification azalma |

### Twin Database Diyagramı
```
┌─────────────────┐      ┌─────────────────┐
│   ClickHouse    │◄────►│  TimescaleDB    │
│ (Ana Canlı DB)  │      │ (OLTP/Streaming)│
└─────────────────┘      └─────────────────┘
        │                         │
        ▼                         ▼
┌─────────────────┐      ┌─────────────────┐
│     Neo4j       │      │     Milvus      │
│ (Knowledge KG)  │      │ (Vector Store)  │
└─────────────────┘      └─────────────────┘
        │                         │
        └────────────┬────────────┘
                   ▼
        ┌─────────────────┐
        │ Delta Lake+Hudi │
        │ (Offline Store) │
        └─────────────────┘
```

## 2.4 Veri İşleme Pipeline ve CDC

### Ana Veri Akışı
```
API → Kafka → Flink Processing → 
    ├── ClickHouse (1M/s ingestion)
    ├── TimescaleDB (real-time queries)
    ├── Redis (online store)
    └── Delta Lake + Hudi (offline store)
```

### CDC Pipeline
```
ClickHouse MV → Kafka Engine → Flink → Feast
    ├── Redis (Lua CAS versioning)
    └── Delta Lake (MERGE INTO)

Neo4j CDC → Debezium → Kafka → Flink → GNN
```

## 2.5 DataPlant Mimarisi (V1 BLUEPRINT)

### 🔴 EKSİK 1 ÇÖZÜMÜ: Coverage Yönetimi

**KARAR:** Strategy Pattern ile imputation + masking

```
┌─────────────────────────────┐
│  API Adapters               │ → CanonicalMatch (w/ coverage_mask)
│  + Global Rate Limiter      │ → Redis Lua Token Bucket
├─────────────────────────────┤
│  ConflictResolver           │ → Priority Failover (Master/Slave)
│  + Monotonicity Check       │ → Redis StateStore + LRU fallback
├─────────────────────────────┤
│  CoverageManager            │ → PolicyRegistry (impute/skip)
│  + Strategy Pattern         │ → freshness_score calculation
└─────────────┬───────────────┘
              ▼
         Kafka Topics
        (CloudEvents)
```

### CoverageInfo Veri Yapısı
```python
@dataclass
class CoverageInfo:
    masked: List[str] = field(default_factory=list)          # Veri yok, skip
    imputed: Dict[str, float] = field(default_factory=dict)  # Dolgu değerleri
    confidence: Dict[str, float] = field(default_factory=dict)  # Güven skorları
```

### Imputation Stratejileri ve Güven Skorları

| Strateji | Kullanım | Confidence Formül |
|----------|----------|-------------------|
| **LeagueAvgImputer** | xG lig ortalaması | `c = clamp((1 - std/avg) * sqrt(n/min_n), 0, 0.9)` |
| **ConstantImputer** | Possession %50 default | `c = 0.1` (sabit) |
| **EWMA Imputer** | Adaptif average | `c = clamp(1 - mae/avg, 0, 0.7)` |

**Kritik Kural:** `confidence < 0.3` → alan `masked` listesine taşınır, model yok sayar

### Coverage Manager Kodu
```python
strategies = {
    'xg': LeagueAvgImputer(window=5),       # Lig ortalaması
    'possession': ConstantImputer(0.5)      # %50 default
}

def apply_coverage(match):
    for field, strat in strategies.items():
        if match.coverage_mask[field]:      # Veri yoksa
            match[field] = strat.impute(match)
            match.metadata['imputed'].append(field)
            
            # Güven skoru hesapla
            confidence = strat.calculate_confidence(match)
            if confidence < 0.3:
                # Düşük güvenli veri → masked
                match.coverage_mask[field] = True
                match.metadata['imputed'].remove(field)
```

### 🔴 EKSİK 2 ÇÖZÜMÜ: Multi-API Koordinasyonu

**KARAR:** Priority Failover (Master/Slave) + Monotonicity Check

```python
class ConflictResolver:
    def resolve(self, match_id):
        # Primary API fail → Secondary
        primary_data = primary_api.fetch(match_id)
        
        # Monotonicity Check (gecikmiş/hatalı veri reddi)
        last_known = redis_state.get(match_id) or lru_cache.get(match_id)
        
        if (primary_data.score < last_known.score or 
            primary_data.minute < last_known.minute):
            return last_known  # Geçersiz veri reddedildi
        
        # 2s reconciliation window + hysteresis
        if abs(primary_data.ts - secondary_data.ts) <= 2:
            return max(primary_data, secondary_data, key=lambda x: x.ts)
        
        # Consensus başarısız → Failover
        if primary_misses >= 2:
            demote_primary()
            return secondary_api.fetch(match_id)
        
        return primary_data
```

**StateStore CircuitBreaker Fallback:**
- Redis erişilemezse → In-memory LRU cache (max 1000 match)
- Cache: Caffeine (write-through, 5 min TTL)

### Global Rate Limiter (Redis Token Bucket)

```python
@RateLimit(key="api_v3", limit=300, fallback=LocalBucket)
def fetch_match(match_id):
    lua = """
    if redis.call('get', KEYS[1]) > 0 then 
        return redis.call('decr', KEYS[1]) 
    else 
        return -1 
    end
    """
    if redis.eval(lua, 1, f"rate:{provider.id}") < 0:
        return switch_provider()  # Limit doldu → yedek API
    return provider.get_match(match_id)
```

**Fallback:** Redis erişilemezse → lokal TokenBucket (limit / instance_count)

### 🔴 EKSİK 3 ÇÖZÜMÜ: Data Freshness

**KARAR:** Feature bazlı TTL + exponential freshness score

```python
freshness_score = exp(-(now - event_time) / feature_ttl)

if freshness_score < 0.3:
    # Stale veri → IntelligencePlant otomatik SKIP
    return None
```

**CloudEvents Entegrasyonu:**
```json
{
  "specversion": "1.0",
  "type": "match.update",
  "data": { "xg": 1.2 },
  "extensions": {
    "imputedfields": "xg,possession",
    "freshness": 0.85,
    "confidence_map": {"xg": 0.6, "possession": 0.1},
    "traceparent": "00-abc123...",
    "schema_version": "v1.2"
  }
}
```

### DataPlant Final Kodu
```python
class DataPlant:
    def process(self, match_id):
        # 1. Adapters: Rate limited fetch
        raw_data = self.adapters.fetch(match_id)
        
        # 2. ConflictResolver: Monotonic + consensus
        resolved = self.resolver.resolve(raw_data)
        
        # 3. CoverageManager: Imputation + confidence
        covered = self.coverage.apply_coverage(resolved)
        
        # 4. Freshness check
        covered.extensions['freshness'] = self.calculate_freshness(covered)
        
        # 5. Kafka produce (CloudEvents)
        self.kafka.produce(topic='football.live.updates', data=covered)
```

---

# 📌 BÖLÜM 3: ZEKA KATMANI

## 3.1 HRL (Hierarchical Reinforcement Learning)

```
┌─────────────────────────────────────────────┐
│  MANAGER AGENT (UCB Stratejisi)            │
│  State: [bütçe, risk_score, portföy_return, │
│          sub_agent_perf, market_volatility] │
│  Action: Bütçe dağıtımı (Live/Prematch)     │
└─────────────┬───────────────────────────────┘
              │
    ┌─────────┴─────────┐
    ▼                   ▼
┌───────────────┐   ┌───────────────┐
│ LIVE WORKER   │   │ PREMATCH WORKER│
│ LSTM + PPO    │   │ DQN            │
│ Momentum odak │   │ İstatistik odak│
└───────────────┘   └───────────────┘
```

### Manager Agent Performans Güncelleme
**KARAR:** `collections.deque(maxlen=10)` kullanımı (O(1) vs slicing O(k))

```python
from collections import deque

class ManagerAgent:
    def __init__(self):
        self.roi_history = deque(maxlen=10)  # Sabit bellek, O(1)
    
    def update_performance(self, latest_roi):
        self.roi_history.append(latest_roi)  # Auto-evict en eski
        state.sub_agent_performance = np.mean(self.roi_history)
```

**Gerekçe:** KServe pod'larında %20 throughput artışı

## 3.2 Model Mimarisi: Graph-LSTM + LSTM-State-Space + TFT

### Entegre Mimari
```
┌─────────────────────────────────────────────┐
│  1. GRAPH-LSTM (Encoder)                    │
│     - GNN: Oyuncu/takım ilişkileri          │
│     - Global Attention Pooling               │
│     - Spatial-Temporal Embedding             │
├─────────────────────────────────────────────┤
│  2. LSTM-STATE-SPACE (Core)                 │
│     - Non-lineer dinamikler                  │
│     - Futbolun stokastik doğası              │
│     - Flink + ONNX real-time inference       │
├─────────────────────────────────────────────┤
│  3. TFT (Decoder)                           │
│     - Variable Selection Network             │
│     - Interpretability (Attention weights)   │
│     - RL state'e attention ekleme            │
└─────────────────────────────────────────────┘
```

### Graph-LSTM Encoder
```python
class GraphLSTMEncoder(nn.Module):
    def forward(self, x, edge_index, batch):
        # 1. GNN spatial encoding
        x_graph = self.gnn(x, edge_index)
        
        # 2. Global Attention Pooling
        pooled = self.attention_pool(x_graph, batch)
        
        # 3. Sequence formatı (Batch, Seq, Embed)
        pooled_seq = pooled.view(batch_size, seq_len, -1)
        
        # 4. LSTM temporal encoding
        lstm_out, (h_n, c_n) = self.lstm(pooled_seq)
        
        return lstm_out, h_n
```

### LSTM-State-Space Core
```python
class LSTMStateSpaceCore(nn.Module):
    def __init__(self, input_dim, hidden_dim, state_dim):
        super().__init__()
        self.lstm = nn.LSTM(input_dim, hidden_dim, bidirectional=True)
        self.state_space = StateSpaceModel(input_dim, state_dim)
        
        # Bidirectional Cross-attention
        self.cross_attn_lstm_to_ss = nn.MultiheadAttention(embed_dim=hidden_dim * 2, num_heads=8)
        self.cross_attn_ss_to_lstm = nn.MultiheadAttention(embed_dim=state_dim, num_heads=8)
```

### TFT Decoder
```python
class TFTDecoder(nn.Module):
    def forward(self, lstm_output, static_features):
        # Variable Selection Network
        selected_static, static_weights = self.vsn_static(static_features)
        
        # Interpretable Multi-Head Attention
        output, attention_weights = self.attention(lstm_output)
        
        # RL agent'a hem tahmin hem interpretability
        return output, attention_weights
```

## 3.3 Feature Engineering (Feast Integration)

```python
# Offline features - TTL: 365 gün
match_stats_fv = FeatureView(name="match_statistics", ttl=timedelta(days=365))

# Online features - TTL: 30 saniye
live_odds_fv = FeatureView(name="live_odds", ttl=timedelta(seconds=30))

# GNN embeddings (Protobuf/Bytes)
graph_embedding_fv = FeatureView(
    name="graph_embeddings",
    ttl=timedelta(minutes=5),
    schema=Schema(fields=[Field(name="embedding", dtype=Bytes)])
)

# Confidence features (EKSİK 1 çözümü)
# Feast'te confidence_* prefix ile saklanır
confidence_fv = FeatureView(
    name="confidence_scores",
    ttl=timedelta(minutes=1),
    schema=Schema(fields=[
        Field(name="confidence_xg", dtype=Float32),
        Field(name="confidence_possession", dtype=Float32)
    ])
)
```

### Model Input Pipeline (Confidence Entegrasyonu)

```python
# Feast'ten feature + confidence çekimi
features = feast_client.get_online_features(
    features=['match_statistics:xg', 
              'confidence_scores:confidence_xg'],
    entity_rows=[{'match_id': match_id}]
).to_dict()

# PyTorch Input: [Feature_Value, Confidence_Score]
vals = torch.tensor([features['xg']])
conf = torch.tensor([features['confidence_xg']])

# Masking: confidence < 0.3 veya coverage_mask=True
mask = (conf < 0.3) | coverage_mask['xg']

# MaskedTensor oluşturma
x = torch.stack([vals, conf], dim=-1)  # [Batch, Features, 2]
masked_vals = MaskedTensor(x[..., 0], mask)
confidences = x[..., 1]

# Model forward pass
output = model(masked_vals, confidences)
```

---

# 📌 BÖLÜM 4: RİSK KATMANI

## 4.1 RDQL (Risk-constrained DQL)

```python
class RDQL_Agent:
    def compute_loss(self, batch):
        # Standard Q-loss
        q_loss = F.mse_loss(q_values, target_q)
        
        # Cost constraint (risk limiti)
        constraint_violation = F.relu(cost_values.mean() - self.risk_threshold)
        
        # Lagrangian relaxation
        total_loss = q_loss + self.lambda_multiplier * constraint_violation
        
        # Dual ascent (lambda güncelleme)
        self.lambda_multiplier += 0.01 * constraint_violation.item()
        
        return total_loss
```

## 4.2 Risk Metrikleri

| Metrik | Formül | Kullanım |
|--------|--------|----------|
| VaR (5%) | `percentile(returns, 5%)` | Günlük kayıp limiti |
| CVaR | `mean(returns[returns <= VaR])` | Worst-case analizi |
| Max Drawdown | `min((equity - peak) / peak)` | Toplam kayıp limiti |
| Sharpe Ratio | `sqrt(252) * mean(excess) / std(excess)` | Risk-adjusted return |

## 4.3 Sabit Risk Limitleri

```python
RISK_LIMITS = {
    "max_single_bet": 0.05,      # Bankroll max %5
    "max_daily_loss": 0.10,      # Günlük max %10 kayıp
    "max_weekly_loss": 0.20,     # Haftalık max %20 kayıp
    "min_odds": 1.20,
    "max_odds": 10.0,
    "max_open_bets": 10
}
```

## 4.4 Stake Sizing: CVaR-Constrained Thompson Sampling

### Fractional Kelly (Temel)
```python
def calculate_stake(confidence, kelly_fraction, bankroll):
    kelly_stake = kelly_fraction * confidence * bankroll
    max_stake = 0.05 * bankroll
    return min(kelly_stake, max_stake)
```

### CVaR-Constrained Thompson Sampling (Gelişmiş)

```python
from scipy.stats import beta as beta_dist

def constrained_thompson_sampling(priors, cvar_limit=0.05, bankroll=1000):
    """
    - priors: [(alpha, beta), ...] her arm için
    - cvar_limit: Min kabul CVaR (0.05 = %5)
    """
    # Beta dağılımından sample
    samples = [beta_dist.rvs(alpha, beta_param) for alpha, beta_param in priors]
    
    # CVaR filtresi: %5 VaR kontrolü
    valid_actions = [
        i for i in range(len(samples)) 
        if np.percentile([samples[i]], 5) >= cvar_limit
    ]
    
    if not valid_actions:
        return None, 0  # Hiçbir aksiyon uygun değil - bekle
    
    # En yüksek expected value
    best_action = max(valid_actions, key=lambda i: samples[i])
    
    # CVaR-constrained stake
    stake = min(
        bankroll * 0.05,                      # Max %5 single bet
        bankroll * samples[best_action] * 0.3  # %30 fractional
    )
    
    return best_action, stake
```

---

# 📌 BÖLÜM 5: ÖDÜL FONKSİYONU

```python
def compute_reward(state, payout, stake):
    # 1. ROI
    roi = (payout - stake) / (stake + 1e-6)
    
    # 2. Risk Ayarlı Getiri (Sharpe Proxy)
    risk_adjusted_roi = roi / (state.market_volatility * state.risk_score + 1e-6)
    
    # 3. Bütçe Koruması Cezası
    budget_penalty = 0.1 * max(0, 0.8 - state.butce_kalan / state.baslangic_butcesi)
    
    # 4. Dinamik Break-Even Bonusu
    break_even = 1.0 / (state.avg_odds + 1e-6)
    performance_bonus = 0.2 * (state.last_10_win_rate - break_even)
    
    return risk_adjusted_roi - budget_penalty + performance_bonus
```

---

# 📌 BÖLÜM 6: EĞİTİM STRATEJİSİ

## 6.1 Curriculum Learning

```
Phase 1: Sadece Prematch (basit) → Win rate > 55%
    ↓
Phase 2: Sadece Live (orta) → Win rate > 52%
    ↓
Phase 3: Combined (zor) → Win rate > 50%
    ↓
Phase 4: Full HRL (çok zor)
```

## 6.2 Distributed Training
- **Ray RLlib:** Distributed PPO/DQN eğitimi
- **Optuna HPO:** Hyperparameter tuning

## 6.3 Backtesting
- **Walk-forward validation**
- **Shadow testing:** %10 trafik yeni modele

---

# 📌 BÖLÜM 7: PRODUCTION MİMARİSİ

## 7.1 Docker Stack

```yaml
services:
  adapter:          # FastAPI + Circuit Breaker
  app:              # HRL Agents
  clickhouse:       # 1M/s ingestion
  timescale:        # OLTP
  redis:            # Cache + Online Store
  neo4j:            # Knowledge Graph
  milvus:           # Vector Store
  kafka:            # Real-time streaming
  flink:            # CDC Processing
  prometheus:       # Metrics
  grafana:          # Visualization
```

## 7.2 Model Serving: KServe

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: hrl-agent-live
spec:
  predictor:
    pytorch:
      storageUri: "s3://models/hrl-agent-live/v1"
      resources:
        limits:
          nvidia.com/gpu: 1
  transformer:
    containers:
      - name: feature-processor
        image: feast-feature-processor:latest
        env:
          - name: FEAST_ONLINE_STORE
            value: "redis://redis:6379"
```

### Canary Deployment
```yaml
spec:
  canaryTrafficPercent: 10  # %10 yeni modele
  predictor:
    canary:
      pytorch:
        storageUri: "s3://models/hrl-agent-live/v2"
```

---

# 📌 BÖLÜM 8: V1 BLUEPRINT - EKSİKLERİN ÇÖZÜMÜ

## 🔴 EKSİK 4: Cold Start Problemi

**KARAR:** BootstrapPlant + TGN/GraphSAGE Knowledge Distillation

### Asenkron Event-Driven Akış
```
1. CoverageManager: "Unknown Team ID" tespit
    ↓
2. Kafka: `team.coldstart` eventi
    ↓
3. BootstrapPlant: TGN embedding üretimi (Teacher)
    ↓
4. Knowledge Distillation: GraphSAGE (Student) eğitimi
    ↓
5. Milvus/Feast: Embedding yazma
    ↓
6. DataPlant: Sonraki eventte veriyi bulur
```

### BootstrapPlant Kodu
```python
def bootstrap_team(team_id):
    # Neo4j graph similarity
    neighbors = neo4j.run("""
        MATCH (t:Team {id: $team_id})-[:SIMILAR]->(n) 
        RETURN n LIMIT 5
    """, team_id=team_id)
    
    # Milvus vector search
    features = milvus.search(neighbors, k=5)
    
    # TGN Teacher: Temporal embedding
    tgn_embedding = tgn_model(features, temporal_edges)
    
    # GraphSAGE Student: Distillation
    student_embedding = graphsage_model(features)
    distillation_loss = F.mse_loss(student_embedding, tgn_embedding.detach())
    
    # Feast'e yaz (is_cold_start flag)
    feast.write_features({
        'team_id': team_id,
        'embedding': student_embedding.numpy(),
        'is_cold_start': True,
        'confidence': 0.5  # Cold start düşük güven
    })
    
    return weighted_average(features, weights='similarity')
```

**Çıkarım Stratejisi:**
- **Gece Batch:** TGN teacher model ile embedding üretimi
- **Canlı Sistem:** GraphSAGE student model (O(1) inference)
- **Fallback:** Student uncertainty > 0.3 → Rule-Based istatistik model
- **Edge Case:** Tüm fallback fail → SKIP

## 🔴 EKSİK 5: Model Uncertainty Propagation

**KARAR:** Monte Carlo Dropout + Confidence Tensor

```python
class BNNWrapper(nn.Module):
    def forward(self, x):
        with torch.no_grad():
            # MC-Dropout: 30 sample ile uncertainty tahmini
            preds = [self.model(x) for _ in range(30)]
            
            mean = torch.mean(torch.stack(preds), dim=0)
            uncertainty = torch.std(torch.stack(preds), dim=0)
            
            # Manager Agent'a input
            if uncertainty > 0.4:
                return {'action': 'skip', 'uncertainty': uncertainty}
            
            return {'prediction': mean, 'uncertainty': uncertainty}
```

**IntelligencePlant Entegrasyonu:**
```python
# Freshness + Uncertainty check
if freshness_score < 0.3 or model_uncertainty > 0.4:
    return 'SKIP'
```

## 🔴 EKSİK 6: Event Schema ve Versioning

**KARAR:** CloudEvents standardı + Schema Registry

```json
{
  "specversion": "1.0",
  "type": "football.match.update",
  "source": "/football/api",
  "id": "sha1(match_id, seq)",
  "time": "2025-01-01T12:00:00Z",
  "datacontenttype": "application/json",
  "dataschema": "https://schema.football/v1.2",
  "data": {
    "match_id": "12345",
    "status": "ongoing",
    "coverage": { "xg": 0.5, "da": 3 }
  },
  "extensions": {
    "imputedfields": "xg,possession",
    "freshness": 0.85,
    "confidence_map": {"xg": 0.6},
    "traceparent": "00-abc123...",
    "schema_version": "v1.2",
    "validation_status": "valid"
  }
}
```

**Flink Filtering:**
```sql
SELECT * FROM match_updates
WHERE 
    freshness > 0.3 
    AND validation_status = 'valid'
    AND schema_version IN ('v1.2', 'v1.1')  -- Backward compatibility
```

## 🔴 EKSİK 11: Graceful Degradation

### Circuit Breaker Entegrasyon Matrisi

| Bileşen | Threshold | Timeout | Fallback |
|---------|-----------|---------|----------|
| **DataPlant** | 3 failures | 30s | CanonicalMatch (stale OK) |
| **IntelligencePlant** | 2 failures | 10s | Student → Redis → Rule-Based → Skip |
| **FeatureStore (Feast)** | 5 failures | 5s | Computed on-the-fly |
| **Kafka** | 1 failure | N/A | Retry with exponential backoff |
| **StateStore (Redis)** | 3 failures | 15s | In-memory Caffeine LRU cache (1000 match) |
| **CoverageManager** | 2 failures | 10s | Default imputation (all=0.1 confidence) |

### Timeout Budget Executor

```python
def execute_with_budget(steps, deadline):
    """
    Her step için SLA tahsis et, budget tükendikçe fallback geç
    """
    for step in steps:
        remaining = deadline - time.time()
        if remaining <= 0:
            break  # Budget tükendi
        
        try:
            return step.run(timeout=min(step.sla, remaining))
        except TimeoutError:
            budget -= step.sla
            continue  # Sonraki fallback'e geç
    
    # Tüm fallback'ler fail → safe_mode
    return "SKIP"
```

### Degradation Ladder (IntelligencePlant)

```python
from pybreaker import CircuitBreaker

cb = CircuitBreaker(
    fail_max=2,
    timeout=60,
    state_storage=RedisCircuitBreakerStorage(host="redis", port=6379),
    fallback=graceful_degrade
)

def graceful_degrade(match_id):
    """
    4 Aşamalı Fallback Ladder:
    1. Student Model (GraphSAGE)
    2. Redis Cache (Last known state)
    3. Rule-Based (Heuristic istatistik)
    4. Skip Bet (Güvenli liman)
    """
    # 1. Student Model
    try:
        pred = student_model.predict(match_id, timeout=5)
        if pred.uncertainty < 0.3:
            return pred
    except:
        pass
    
    # 2. Redis Cache
    try:
        cached = redis.get(f"last_prediction:{match_id}")
        if cached and time.time() - cached.ts < 300:  # 5 min TTL
            return cached
    except:
        pass
    
    # 3. Rule-Based Model
    try:
        baseline = baseline_stats_model.predict(match_id)
        return baseline
    except:
        pass
    
    # 4. Skip (Tüm sistemler fail)
    logger.critical(f"All fallbacks failed for {match_id}, entering safe_mode")
    return {"action": "SKIP", "reason": "all_systems_down"}
```

### Safe Mode Mekanizması

```python
class SystemState:
    modes = ["NORMAL", "DEGRADED", "SAFE_MODE"]
    current = "NORMAL"
    
    def check_and_transition(self):
        # Circuit Breaker durumlarını kontrol et
        open_breakers = [
            cb for cb in all_circuit_breakers 
            if cb.state == "OPEN"
        ]
        
        if len(open_breakers) >= 3:
            # 3+ bileşen fail → SAFE_MODE
            self.current = "SAFE_MODE"
            self.halt_all_betting()
            self.notify_pagerduty()
        elif len(open_breakers) >= 1:
            self.current = "DEGRADED"
```

## 🔴 EKSİK 12: Configuration Management

**KARAR:** Consul/Etcd dinamik config + Feature Flags

```python
# Runtime config (Consul)
threshold = consul.get("circuit/failure_threshold", default=5)
kelly_fraction = consul.get("risk/kelly_fraction", default=0.75)

# Feature Flags (LaunchDarkly - V2)
if feature_flag.is_enabled("use_tgn_teacher"):
    model = tgn_model
else:
    model = graphsage_model

# Secret Management (HashiCorp Vault)
api_key = vault.read("secret/data/api-football")["api_key"]
```

**Edge Case:** Config server erişilemezse
```python
try:
    config = consul.get_all()
except ConnectionError:
    # Fail-safe: Yerel en muhafazakar ayarlar
    config = json.load(open("safe_mode_config.json"))
    logger.warning("Using local safe_mode_config.json")
```

---

# 📌 BÖLÜM 9: OBSERVABİLİTY (EKSİK 8)

## Metrik Katalogu

### Business KPI'lar (Prometheus)

```yaml
metrics:
  - name: prediction_confidence
    help: "Model tahmin güven seviyesi"
    type: gauge
    labels: [model, league]
  
  - name: action_distribution
    help: "Alınan aksiyonların dağılımı"
    type: histogram
    buckets: [0, 0.2, 0.5, 1.0, 2.0, 5.0]
  
  - name: roi_per_hour
    help: "Saatlik ROI metriği"
    type: gauge
  
  - name: circuit_state_change
    help: "Circuit Breaker durum değişiklikleri"
    type: counter
    labels: [component, state]
  
  - name: fallback_rate
    help: "Fallback kullanım oranı"
    type: counter
    labels: [component, fallback_step]
  
  - name: imputation_confidence_avg
    help: "Ortalama imputation güven skoru"
    type: gauge
    labels: [strategy]
```

### Circuit Breaker Observability

```python
# Circuit Breaker olaylarını loglama
breaker.on_open = lambda: metrics.inc(
    "circuit_state_change", 
    labels={"service": "inference", "state": "open"}
)

breaker.on_half_open = lambda: metrics.inc(
    "circuit_state_change",
    labels={"service": "inference", "state": "half_open"}
)

# Fallback metriği
def graceful_degrade(match_id):
    for step_name, step_fn in fallback_ladder:
        try:
            result = step_fn(match_id)
            metrics.inc("fallback_rate", labels={
                "component": "intelligence",
                "fallback_step": step_name
            })
            return result
        except:
            continue
```

### Grafana Dashboard

**SLO Compliance Widget:**
```yaml
dashboard:
  - name: "Betting System ROI"
    panels:
      - title: "SLO: Freshness > 0.3 (Target: 95%)"
        query: "avg(freshness_score) > 0.3"
        threshold: 0.95
        alert: PagerDuty
      
      - title: "SLO: Fallback Rate < 10% (Target: 90%)"
        query: "rate(fallback_rate[5m]) < 0.10"
        threshold: 0.90
        alert: Slack
```

### Distributed Tracing (V2 Roadmap)

**CloudEvents traceparent:**
```json
"extensions": {
  "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba90200-01"
}
```

**Jaeger Integration:**
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger import JaegerExporter

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("dataplane.process"):
    with tracer.start_as_current_span("adapter.fetch"):
        raw_data = adapter.fetch(match_id)
    
    with tracer.start_as_current_span("resolver.resolve"):
        resolved = resolver.resolve(raw_data)
```

---

# 📌 BÖLÜM 10: CI/CD VE TESTING

## Zorunlu Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - lint
  - build
  - deploy

test:
  script:
    - pytest --coverage --cov-fail-under=80
    - mypy src/ --strict
  artifacts:
    reports:
      coverage: coverage.xml

lint:
  script:
    - black --check src/
    - flake8 src/
    - pylint src/

shadow_test:
  stage: deploy
  script:
    - deploy_canary --traffic-percent=10
    - wait_for_metrics --duration=1h
    - if metrics.sharpe > 0.7: promote_to_production
```

---

# 📌 BÖLÜM 11: V1 BLUEPRINT SON HAL

## Nihai Mimari Özet

```
┌─────────────────────────────────────────────────────────────┐
│                      EVENT BUS (Kafka)                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
    ┌─────────────────┼─────────────────┐
    ▼                 ▼                 ▼
┌──────────┐   ┌──────────────┐   ┌─────────────┐
│DataPlant │   │IntelligencePlnt│   │BootstrapPlnt│
│          │   │                │   │             │
│ Adapter  │   │ GraphSAGE      │   │ TGN Teacher │
│ Resolver │   │ (Student)      │   │ Distill     │
│ Coverage │   │ MC-Dropout     │   │ Neo4j/Milvus│
└────┬─────┘   └────┬───────────┘   └─────────────┘
     │              │
     ▼              ▼
┌─────────────────────────────┐
│   Feast Feature Store       │
│   - Online: Redis           │
│   - Offline: Delta Lake     │
│   - Confidence_* features   │
└────────────┬────────────────┘
             │
             ▼
┌─────────────────────────────┐
│   KServe Inference          │
│   - Canary: 10% traffic     │
│   - Triton: FP16 optimize   │
└─────────────────────────────┘
```

## Kritik Başarı Faktörleri

### Coverage Management (EKSİK 1)
✅ Strategy Pattern ile imputation  
✅ Confidence scores: LeagueAvg (0.9), Constant (0.1), EWMA (0.7)  
✅ Threshold 0.3 altı → masked  
✅ CloudEvents extensions → model input [value, confidence]

### Multi-API Koordinasyonu (EKSİK 2)
✅ Priority Failover (Master/Slave)  
✅ Monotonicity Check (gecikmiş veri reddi)  
✅ Redis StateStore + Caffeine LRU fallback  
✅ Rate Limiter: Redis Token Bucket + lokal fallback

### Data Freshness (EKSİK 3)
✅ `freshness_score = exp(-age / ttl)`  
✅ Score < 0.3 → otomatik SKIP  
✅ Feature bazlı TTL (live:30s, offline:365d)

### Cold Start (EKSİK 4)
✅ BootstrapPlant: TGN/GraphSAGE distillation  
✅ Asenkron event-driven (`team.coldstart`)  
✅ Milvus vector search + Neo4j graph similarity  
✅ Student uncertainty > 0.3 → Rule-Based fallback

### Model Uncertainty (EKSİK 5)
✅ Monte Carlo Dropout (30 samples)  
✅ Uncertainty > 0.4 → SKIP  
✅ Manager Agent'a uncertainty input

### Event Schema (EKSİK 6)
✅ CloudEvents standardı  
✅ Schema versioning (v1.2 backward compatible)  
✅ Extensions: freshness, confidence_map, traceparent, validation_status

### Graceful Degradation (EKSİK 11)
✅ Circuit Breaker matrisi (6 bileşen)  
✅ Timeout Budget Executor  
✅ 4 aşamalı fallback ladder  
✅ Safe Mode (3+ breaker open)

### Config Management (EKSİK 12)
✅ Consul/Etcd runtime config  
✅ Vault secret management  
✅ safe_mode_config.json fallback

### Observability (EKSİK 8)
✅ Business KPI metrikleri (Prometheus)  
✅ Circuit Breaker event tracking  
✅ SLO compliance dashboard (Grafana)  
✅ PagerDuty alerting (fallback > 10%)

### CI/CD
✅ pytest --coverage (min 80%)  
✅ mypy --strict  
✅ Shadow testing %10 traffic  
✅ Canary deployment gates

---

# 📌 BÖLÜM 12: V2 ROADMAP (Ertelenenler)

## V2'de Ele Alınacak Eksiklikler

| EKSİK | Konu | V2 Çözüm Önerisi |
|-------|------|------------------|
| **EKSİK 7** | State Recovery | Kafka-based checkpointing + event replay |
| **EKSİK 8 (ek)** | Distributed Tracing | Jaeger entegrasyonu + correlation ID |
| **EKSİK 9** | Feature Dependency | DAGScheduler + runtime fallback_chain |
| **EKSİK 10** | Time Synchronization | Cross-timezone correlation + NTP sync |
| **EKSİK 12 (ek)** | Feature Flags | LaunchDarkly integration |

## Plant Specifications

Her tesis için `plant_spec.yaml` oluşturulacak:

```yaml
# dataplane_spec.yaml
name: DataPlant
version: v1.0
contracts:
  input:
    - type: APIResponse
      schema: api_response_v1.json
  output:
    - type: CloudEvent
      schema: cloudevents_v1.0
  
components:
  - name: APIAdapter
    circuit_breaker:
      fail_max: 3
      timeout: 30
    rate_limiter:
      redis_key: "rate:api_v3"
      limit: 300
  
  - name: ConflictResolver
    state_store:
      primary: redis
      fallback: caffeine_lru
      lru_size: 1000
  
  - name: CoverageManager
    strategies:
      xg: LeagueAvgImputer
      possession: ConstantImputer
    confidence_threshold: 0.3
```

---

# 📊 NİHAİ PERFORMANS HEDEFLERİ

## Sistem Metrikleri

| Metrik | Hedef | Strateji |
|--------|-------|----------|
| **Latency (p99)** | < 60ms | Triton FP16 + Priority Queue |
| **Throughput** | +40% | TensorRT optimization |
| **VRAM (Serving)** | 16Gi | FSDP + CPU offload |
| **VRAM (Training)** | 32Gi | Activation checkpointing + Mixed Precision |
| **Freshness SLO** | > 95% | Auto-skip stale data |
| **Fallback Rate** | < 10% | Robust Circuit Breaker ladder |
| **Coverage (test)** | > 80% | Pytest mandatory |

## ROI Hedefleri

| Aşama | ROI Hedefi | Risk Kontrolü |
|-------|-----------|---------------|
| **Phase 1** (Prematch) | Win rate > 55% | VaR limitleri |
| **Phase 2** (Live) | Win rate > 52% | CVaR-constrained Thompson |
| **Phase 3** (Combined) | Win rate > 50% | Fractional Kelly (0.75) |
| **Phase 4** (Full HRL) | Sharpe > 0.8 | Circuit Breaker gates |

---

# 🎯 SONUÇ

## V1 Blueprint Tamamlandı ✅

**4 Turda Çözülen Eksiklikler:**
1. ✅ Coverage Yönetimi (Strategy Pattern + Confidence)
2. ✅ Multi-API Koordinasyonu (Priority Failover + Monotonic)
3. ✅ Data Freshness (Exponential scoring + auto-skip)
4. ✅ Cold Start (TGN/GraphSAGE Distillation)
5. ✅ Model Uncertainty (MC-Dropout + threshold)
6. ✅ Event Schema (CloudEvents + versioning)
7. ✅ Graceful Degradation (CircuitBreaker matrix + 4-tier fallback)
8. ✅ Observability (Business KPI + SLO dashboard)
9. ✅ Config Management (Consul/Etcd + safe_mode)

**Nihai Sistem Karakteristikleri:**
- **Modüler:** Plant-based architecture
- **Resilient:** 6 bileşende Circuit Breaker
- **Observability:** Prometheus + Grafana + PagerDuty
- **CI/CD:** pytest + mypy + shadow testing
- **Risk-Aware:** CVaR-constrained Thompson + Fractional Kelly

**Implementation Ready:** 
Tüm bileşenler için somut kod, mimari diyagramlar ve entegrasyon noktaları belirlendi. V1 production deployment başlatılabilir.

---

**Manifesto Kaynağı:** 3 münazara transkripti  
**Toplam Token:** 701,387  
**Toplam Maliyet:** $0.70139  
**Tarih:** 01.01.2026  
**Blueprint Versiyonu:** v1.0 (LOCKED)
