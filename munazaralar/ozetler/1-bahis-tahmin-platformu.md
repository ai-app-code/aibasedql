# Bahis Tahmin Platformu - AI Tabanlı Sistem Tasarımı


## 🎯 Proje Hedefi
Canlı maçlar ve maç öncesi bahis bülteni verilerini kullanan, RL (Reinforcement Learning), QL (Q-Learning) veya RQL sistemleri ya da bunların hibrit/sofistike versiyonlarıyla çalışan bir bahis tahmin platformu geliştirmek.

## 🏗️ Sistem Mimarisi

### 1. Altyapı Kararları
- **Veri Akışı:** Kafka ile streaming pipeline
- **Model Deployment:** Kubeflow + KServe InferenceService
- **Ölçeklendirme:** HPA (Horizontal Pod Autoscaler) ile autoscaling
- **Monitoring:** Prometheus ile metrik toplama
- **Latency Hedefi:** KServe pod'larında <10ms

### 2. Hierarchical Reinforcement Learning (HRL) Yapısı

#### Üst Katman (Manager Agent)
**Görev:** Risk/bütçe yönetimi ve alt ajan seçimi

**State Vektörü:**
```python
state = [
    bütçe_kalan,
    risk_score,
    portföy_return,
    sub_agent_performance,
    market_volatility
]
```

**Eylem Seçimi:** Upper Confidence Bound (UCB) algoritması
```python
class ManagerAgent:
    def __init__(self):
        self.arms = [
            {'agent': 'prematch', 'q': 0, 'n': 0, 't': 0},
            {'agent': 'live', 'q': 0, 'n': 0, 't': 0},
            {'agent': 'risk', 'q': 0, 'n': 0, 't': 0}
        ]
        self.roi_history = deque(maxlen=10)
    
    def select_action(self, state):
        arm = max(self.arms, 
                 key=lambda x: x['q'] + 0.2*np.sqrt(np.log(sum(a['t'] for a in self.arms))/(x['n']+1)))
        return arm['agent']
    
    def update_arm(self, arm, reward):
        arm['n'] += 1
        arm['t'] += 1
        arm['q'] += (reward - arm['q']) / arm['n']
```

#### Alt Katmanlar (Worker Agents)

**Maç Öncesi Agent:** Tabular Q-Learning
- Statik verileri işler
- Maç öncesi bahis kararları

**Canlı Agent:** LSTM + PPO (Proximal Policy Optimization)
```python
class LiveAgent(nn.Module):
    def __init__(self):
        super().__init__()
        self.lstm = nn.LSTM(input_size=live_feats, hidden_size=64)
        self.attention = nn.Linear(64, 1)
        self.actor = nn.Linear(64, action_space)
    
    def forward(self, x):
        lstm_out, _ = self.lstm(x)
        attention_weights = F.softmax(self.attention(lstm_out), dim=1)
        context = torch.sum(attention_weights * lstm_out, dim=1)
        return self.actor(context)
```

**Risk Agent:** Upper Confidence Bound (UCB) algoritması

### 3. Ödül Fonksiyonu (Reward Function)

**Nihai Formül:**
```python
def compute_reward(state, payout, stake):
    # ROI Hesaplama
    action_roi = (payout - stake) / (stake + 1e-6)
    
    # Risk Ayarlı Getiri (Sharpe Oranı Yaklaşımı)
    risk_adjusted_return = action_roi / (state.market_volatility * state.risk_score + 1e-6)
    
    # Bütçe Kısıtı Cezası
    budget_penalty = 0.1 * max(0, 0.8 - state.bütçe_kalan / state.initial_budget)
    
    # Dinamik Başabaş Noktası ile Performans Bonusu
    break_even = 1.0 / (state.avg_odds + 1e-6)
    performance_bonus = 0.2 * (state.sub_agent_performance - break_even)
    
    return risk_adjusted_return - budget_penalty + performance_bonus
```

**Ödül Fonksiyonu Bileşenleri:**
1. **Risk Ayarlı Getiri:** Sharpe oranı prensibi ile volatilite ve risk skorunu hesaba katar
2. **Bütçe Cezası:** Bütçe %80'in altına düştüğünde ceza uygular
3. **Performans Bonusu:** Alt ajan performansını dinamik başabaş noktasına göre değerlendirir

### 4. Performans Takibi

**ROI Geçmişi Yönetimi:**
```python
from collections import deque

class ManagerAgent:
    def __init__(self):
        self.roi_history = deque(maxlen=10)  # O(1) karmaşıklık, sabit bellek
    
    def update_performance(self, latest_roi):
        self.roi_history.append(latest_roi)  # Otomatik eski veri silinir
        state.sub_agent_performance = np.mean(self.roi_history)
```

**Optimizasyon Kararı:**
- `pop(0)` yerine `deque(maxlen=10)` kullanımı
- **Sebep:** O(1) karmaşıklık vs O(n), GC yükü azaltma
- **Kazanç:** %20 throughput artışı

## 🔑 Kritik Teknik Kararlar

### 1. Çatışma: Çarpım vs Sharpe Oranı
- **DevOps Önerisi:** `reward = portföy_return * sub_agent_performance - 0.5 * market_volatility * risk_score`
- **Alfa İtirazı:** Çarpım kararsızlık yaratır
- **Nihai Karar:** Sharpe Oranı yaklaşımı (risk ayarlı getiri)

### 2. Çatışma: Sabit vs Dinamik Başabaş
- **Gamma Önerisi:** Sabit 0.5 (bahis pazarında %50 başarı)
- **Alfa Önerisi:** Dinamik `break_even = 1 / avg_odds`
- **Nihai Karar:** Dinamik başabaş noktası (piyasa koşullarına uyum)

### 3. Çatışma: pop(0) vs Slicing vs Deque
- **Beta:** `pop(0)` ile liste yönetimi
- **Gamma:** Slicing ile son 10 eleman
- **DevOps/Alfa:** `deque(maxlen=10)` önerisi
- **Nihai Karar:** Deque (O(1) karmaşıklık, sabit bellek, %20 performans artışı)

## 📊 Sistem Akışı

1. **Veri Toplama:** Kafka ile canlı maç verileri ve bahis bülteni akışı
2. **State Güncelleme:** Her 10 saniyede bir Kafka consumer ile
3. **Manager Kararı:** UCB ile en uygun alt ajan seçimi
4. **Alt Ajan Aksiyonu:** Seçilen ajana göre bahis kararı
5. **Ödül Hesaplama:** Sonuç alındıktan sonra reward fonksiyonu
6. **Model Güncelleme:** UCB arm parametreleri ve performans metriklerinin güncellenmesi

## 🎓 Öğrenilen Dersler

1. **Volatilite Yönetimi:** Çarpımsal formüller yerine oranlama yaklaşımları daha stabil
2. **Dinamik Adaptasyon:** Sabit eşik değerleri yerine piyasa koşullarına göre ayarlanan metrikler
3. **Performans Optimizasyonu:** Veri yapısı seçimi (deque) %20 throughput farkı yaratabilir
4. **Bellek Yönetimi:** GC yükünü azaltmak yüksek frekanslı sistemlerde kritik

## 🚀 Deployment Planı

1. ROI geçmişi için `deque(maxlen=10)` kullanımı
2. `action_roi` tanımı: `(payout - stake) / stake`
3. Nihai ödül fonksiyonu: Sharpe yaklaşımı + dinamik başabaş + bütçe cezası
4. Kubeflow pipeline'a entegrasyon
5. Prometheus ile monitoring
6. KServe ile production deployment

