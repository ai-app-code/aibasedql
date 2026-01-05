# 🌌 GRAVITY - Süper-Rasyonel Dijital Bahis Varlığı
## Kapsamlı Mimari Plan ve Uygulama Yol Haritası

**Oluşturma Tarihi:** 03.01.2026  
**Model:** Claude 4.5 Opus (Anthropic)  
**Agent:** Antigravity (Google DeepMind)  
**Kaynak:** 9 münazara özet dosyası  

---

# 📋 İÇİNDEKİLER

1. [Vizyon ve Felsefe](#1-vizyon-ve-felsefe)
2. [Temel Mimari Kararlar](#2-temel-mimari-kararlar)
3. [Veri Katmanı](#3-veri-katmani)
4. [Dijital İkiz Katmanı](#4-dijital-ikiz-katmani)
5. [Zeka Katmanı](#5-zeka-katmani)
6. [Adaptif Mekanizmalar](#6-adaptif-mekanizmalar)
7. [Risk ve Strateji Katmanı](#7-risk-ve-strateji-katmani)
8. [Üretim Altyapısı](#8-uretim-altyapisi)
9. [Uygulama Yol Haritası](#9-uygulama-yol-haritasi)
10. [Kaynak Etiketleme Matrisi](#10-kaynak-etiketleme-matrisi)

---

# 1. VİZYON VE FELSEFE

## 1.1 Temel Amaç

> **"İnsanın analitik düşünce kapasitesini AŞAN, sürekli yaşayan ve evrilen, süper-rasyonel bir dijital bahis varlığı oluşturmak."**

### ⚠️ KRİTİK DÜZELTME

| ❌ YANLIŞ YORUMLAMA | ✅ DOĞRU ANLAM |
|---------------------|----------------|
| "İnsan taklidi" = Duyguları kopyala | "İnsan taklidi" = Analitik gücü **aş** |
| İrrasyonel, duygusal kararlar | Süper-rasyonel, veri odaklı stratejiler |
| Statik, tek seferlik sistem | **Sürekli yaşayan, öğrenen, adapte olan** varlık |

**Kaynak:** Kullanıcı direktifi + [6-otonom-bahis-ai-sistemi.md]

## 1.2 Sistem Felsefesi: İşletim Sistemi Yaklaşımı

```
┌─────────────────────────────────────────────────────────────┐
│                    GRAVITY OPERATING SYSTEM                  │
├─────────────────────────────────────────────────────────────┤
│  KERNEL: Event Bus (Kafka) + State Machine                  │
├─────────────────────────────────────────────────────────────┤
│  PLANTS (Tesisler):                                         │
│    ├── DataPlant (Veri toplama)                            │
│    ├── IntelligencePlant (Tahmin üretme)                   │
│    ├── StrategyPlant (Kupon oluşturma)                     │
│    └── BootstrapPlant (Cold Start yönetimi)                │
├─────────────────────────────────────────────────────────────┤
│  CONTRACTS: Her tesisin uyması gereken interface            │
│  YENİ ÖZELLİK = YENİ TESİS (mevcut kod değişmez)           │
└─────────────────────────────────────────────────────────────┘
```

**Kaynak:** [9-bigplan-manifestosu.md] Bölüm 1.2

## 1.3 Sistemin Temel Yetenekleri

1. **Maç Değerlendirme:** Lig, puan, form, oyuncu gücü, formasyon analizi
2. **Zihinsel Simülasyon:** Maçları oynanmadan "canlandırma" (GNN + Monte Carlo)
3. **Canlı Adaptasyon:** Maç esnasında Bayesian güncelleme
4. **Kupon Stratejileri:** Tekli, çoklu, sistemli bahisler
5. **Kasa Yönetimi:** Kelly, CVaR, portföy optimizasyonu
6. **Sürekli Evrim:** Meta-öğrenme ile strateji geliştirme

**Kaynak:** [4-canlı-futbol-simülasyon-sistemi.md], [5-rdql-sanal-betting-sistemi.md]

---

# 2. TEMEL MİMARİ KARARLAR

## 2.1 Münazaralardan Çıkan Kritik Uzlaşılar

| Karar Alanı | Uzlaşı | Kaynak |
|-------------|--------|--------|
| Ana Canlı Veritabanı | ClickHouse (1M/s ingestion) | [5-rdql] |
| OLTP Veritabanı | TimescaleDB + Hypertable | [2-production], [3-oracle] |
| Knowledge Graph | Neo4j | [4-canlı-simülasyon], [9-bigplan] |
| Vector Store | Milvus (128-boyutlu embedding) | [4-canlı-simülasyon], [9-bigplan] |
| Feature Store | Feast (Online: Redis, Offline: Delta Lake) | [5-rdql], [9-bigplan] |
| Stream Processing | Apache Flink + Kafka | [4-canlı-simülasyon], [5-rdql] |
| Model Serving | KServe + Triton FP16 | [8-implementasyon], [9-bigplan] |
| Risk Yönetimi | CVaR-Constrained Thompson Sampling | [4-canlı-simülasyon] |
| Stake Sizing | Fractional Kelly (0.75) | [5-rdql], [9-bigplan] |

## 2.2 Twin Database Mimarisi

```
┌─────────────────────────────────────────────────────────────┐
│                    TWIN DATABASE ENGINE                      │
├─────────────────┬───────────────────────────────────────────┤
│  HOT DATABASE   │  ClickHouse                               │
│  (Canlı Veri)   │  • 1M/s tick ingestion                    │
│                 │  • ReplacingMergeTree (idempotent upsert) │
│                 │  • Materialized Views (rollup)            │
│                 │  • TTL → S3 Cold Storage                  │
├─────────────────┼───────────────────────────────────────────┤
│  WARM DATABASE  │  TimescaleDB                              │
│  (OLTP/Query)   │  • Hypertable partitioning                │
│                 │  • Continuous Aggregates (1/5/15 min)     │
│                 │  • 30 gün retention policy                │
├─────────────────┼───────────────────────────────────────────┤
│  COLD DATABASE  │  Delta Lake + Hudi MOR                    │
│  (Arşiv)        │  • Merge-on-Read (%60 write azaltma)     │
│                 │  • Async compaction (5-10 dk)             │
│                 │  • S3 object storage                      │
└─────────────────┴───────────────────────────────────────────┘
```

**Teknoloji Gerekçeleri:**
- ClickHouse vs TimescaleDB çatışması → **Her ikisi de kullanılır** (farklı roller)
- ClickHouse: Yüksek frekanslı yazma (1M/s)
- TimescaleDB: Karmaşık sorgular, OLTP

**Kaynak:** [3-project-oracle-twin-engine.md], [5-rdql-sanal-betting-sistemi.md]

## 2.3 Veri Akış Şeması

```
                    ┌──────────────┐
                    │  API-Football │
                    │     v3        │
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │    Kafka     │
                    │   Topics     │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  ClickHouse   │  │  TimescaleDB  │  │    Redis      │
│  (1M/s live)  │  │  (OLTP)       │  │  (Cache)      │
└───────┬───────┘  └───────────────┘  └───────────────┘
        │
        ▼
┌───────────────┐     ┌───────────────┐
│  Flink CDC    │────▶│    Feast      │
│  Processing   │     │ Feature Store │
└───────────────┘     └───────────────┘
```

**Kaynak:** [9-bigplan-manifestosu.md] Bölüm 2.4

---

# 3. VERİ KATMANI (DataPlant)

## 3.1 Canonical Data Model

```python
@dataclass
class CanonicalMatch:
    # Zorunlu alanlar (her zaman mevcut)
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
    
    # Metadata
    coverage_mask: Dict[str, bool]        # Hangi veri mevcut
    data_staleness: float = 0.0           # Gecikme (saniye)
    
    def get_masked_tensor(self) -> Tuple[torch.Tensor, torch.Tensor]:
        """MaskedTensor formatında state döndürür"""
        features = np.array([...])
        mask = ~np.isnan(features).astype(np.float32)
        return torch.tensor(features), torch.tensor(mask)
```

**Kaynak:** [2-production-ready-architecture.md], [9-bigplan-manifestosu.md]

## 3.2 Coverage Yönetimi (Strategy Pattern)

### Imputation Stratejileri

| Strateji | Kullanım | Confidence Formül | Kaynak |
|----------|----------|-------------------|--------|
| LeagueAvgImputer | xG lig ortalaması | `c = clamp((1 - std/avg) * sqrt(n/min_n), 0, 0.9)` | [9-bigplan] |
| ConstantImputer | Possession %50 | `c = 0.1` (sabit) | [9-bigplan] |
| EWMA Imputer | Adaptif ortalama | `c = clamp(1 - mae/avg, 0, 0.7)` | [9-bigplan] |

### Kritik Kural
```python
if confidence < 0.3:
    # Alan "masked" listesine taşınır, model yok sayar
    match.coverage_mask[field] = True
```

**Kaynak:** [9-bigplan-manifestosu.md] Bölüm 2.5

## 3.3 Data Freshness

```python
freshness_score = exp(-(now - event_time) / feature_ttl)

if freshness_score < 0.3:
    # Stale veri → IntelligencePlant otomatik SKIP
    return None
```

**TTL Değerleri:**
- Live odds: 30 saniye
- Match stats: 5 dakika
- Offline features: 365 gün

**Kaynak:** [9-bigplan-manifestosu.md] Bölüm 2.5

## 3.4 Multi-API Koordinasyonu

```python
class ConflictResolver:
    def resolve(self, match_id):
        # Monotonicity Check (gecikmiş/hatalı veri reddi)
        last_known = redis_state.get(match_id) or lru_cache.get(match_id)
        
        if (primary_data.score < last_known.score or 
            primary_data.minute < last_known.minute):
            return last_known  # Geçersiz veri reddedildi
        
        # 2s reconciliation window
        if abs(primary_data.ts - secondary_data.ts) <= 2:
            return max(primary_data, secondary_data, key=lambda x: x.ts)
        
        return primary_data
```

**Fallback Chain:**
1. Redis StateStore
2. In-memory Caffeine LRU cache (max 1000 match)

**Kaynak:** [9-bigplan-manifestosu.md] Bölüm 2.5

---

# 4. DİJİTAL İKİZ KATMANI

## 4.1 Knowledge Graph (Neo4j)

**İçerik:**
- Takım formasyonları
- Oyuncu sakatlıkları
- Tarihsel maç ilişkileri
- Head-to-head verileri
- Teknik direktör değişimleri

```cypher
// Benzer takım bulma
MATCH (t:Team {id: $team_id})-[:SIMILAR]->(n)
RETURN n LIMIT 5
```

**Kaynak:** [4-canlı-futbol-simülasyon-sistemi.md], [9-bigplan-manifestosu.md]

## 4.2 Vector Store (Milvus)

**İçerik:**
- Oyuncu form vektörleri (128-boyut)
- Takım stil embedding'leri
- Maç durumu embedding'leri

```python
# Vector versioning ile atomic swap
milvus.upsert(
    collection="player_embeddings",
    data=new_vectors,
    version=atomic_increment("vector_version")
)

# Async fetch + Redis cache (TTL 30s)
embeddings = await milvus.search(query_vector, k=5)
```

**Kaynak:** [4-canlı-futbol-simülasyon-sistemi.md]

## 4.3 GNN Tabanlı Simülasyon Motoru

### Graf Yapısı
- **Düğümler (Nodes):** Takımlar ve Oyuncular
- **Kenarlar (Edges):** Pas trafiği, formasyonlar, oyun planı
- **Özellikler (Features):** Oyuncu form vektörleri, takım gücü

### Monte Carlo + Bayesian Update

```python
# Pre-Match: 10.000 olasılık simülasyonu
def pre_match_simulation(team_a, team_b):
    scenarios = monte_carlo_simulate(
        teams=[team_a, team_b],
        iterations=10000,
        features=gnn.extract_features()
    )
    return scenarios  # Olasılık dağılımı

# Canlı Maç: Bayesian güncelleme
def live_update(prior_scenarios, live_event):
    posterior = bayesian_update(
        prior=prior_scenarios,
        likelihood=live_event.probability,
        evidence=live_event.data
    )
    return posterior
```

**Kritik Özellik:** Maç başlamadan zihinsel simülasyon → Canlı veriyle anlık revizyon

**Kaynak:** [4-canlı-futbol-simülasyon-sistemi.md]

---

# 5. ZEKA KATMANI (IntelligencePlant)

## 5.1 Hierarchical Reinforcement Learning (HRL)

```
┌─────────────────────────────────────────────────────────────┐
│              MANAGER AGENT (UCB Stratejisi)                  │
│  State: [bütçe, risk_score, portföy_return,                 │
│          sub_agent_perf, market_volatility]                  │
│  Action: Bütçe dağıtımı (Live/Prematch ajan seçimi)         │
└─────────────────────────┬───────────────────────────────────┘
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
┌─────────────────────┐     ┌─────────────────────┐
│   LIVE WORKER       │     │   PREMATCH WORKER   │
│   LSTM + PPO        │     │   DQN               │
│   Momentum odaklı   │     │   İstatistik odaklı │
│   Attention mech.   │     │   Statik analiz     │
└─────────────────────┘     └─────────────────────┘
```

### Manager Agent (UCB)

```python
class ManagerAgent:
    def __init__(self):
        self.roi_history = deque(maxlen=10)  # O(1) karmaşıklık
        self.arms = ['live', 'prematch', 'risk']
    
    def select_action(self, state):
        # Upper Confidence Bound
        arm = max(self.arms, 
                 key=lambda x: x['q'] + 0.2*np.sqrt(
                     np.log(sum(a['t'] for a in self.arms))/(x['n']+1)
                 ))
        return arm
```

**Kaynak:** [1-bahis-tahmin-platformu.md], [9-bigplan-manifestosu.md]

## 5.2 Entegre Model Mimarisi

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 1: GRAPH-LSTM ENCODER                                │
│  • GNN: Oyuncu/takım ilişkileri (spatial)                   │
│  • Global Attention Pooling                                  │
│  • LSTM: Temporal encoding                                   │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2: LSTM-STATE-SPACE CORE                             │
│  • Non-lineer dinamikler modelleme                          │
│  • Bidirectional Cross-Attention                            │
│  • Residual connection (gradient flow)                       │
├─────────────────────────────────────────────────────────────┤
│  LAYER 3: TFT DECODER (Temporal Fusion Transformer)         │
│  • Variable Selection Network                                │
│  • Hangi feature önemli olduğunu öğrenme                    │
│  • Interpretability (attention weights)                      │
└─────────────────────────────────────────────────────────────┘
```

### Graph-LSTM Encoder

```python
class GraphLSTMEncoder(nn.Module):
    def forward(self, x, edge_index, batch):
        # 1. GNN spatial encoding
        x_graph = self.gnn(x, edge_index)
        
        # 2. Global Attention Pooling
        pooled = self.attention_pool(x_graph, batch)
        
        # 3. LSTM temporal encoding
        lstm_out, (h_n, c_n) = self.lstm(pooled_seq)
        
        return lstm_out, h_n
```

**Kaynak:** [5-rdql-sanal-betting-sistemi.md], [9-bigplan-manifestosu.md]

### TFT Decoder

```python
class TFTDecoder(nn.Module):
    def forward(self, lstm_output, static_features):
        # Variable Selection Network
        selected, weights = self.vsn_static(static_features)
        
        # Interpretable attention
        output, attention_weights = self.attention(lstm_output)
        
        # RL agent'a hem tahmin hem interpretability
        return output, attention_weights
```

**Kaynak:** [5-rdql-sanal-betting-sistemi.md]

## 5.3 Üç Katmanlı Modüler Mimari (Production)

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 1: LightGBM-Quantile (Preprocessing)                 │
│  → CAS varyans daraltma, feature extraction                  │
│  → q_dynamic (0.6-0.9) output                               │
├─────────────────────────────────────────────────────────────┤
│  LAYER 2: HyperNetworks (Core)                              │
│  → Dinamik aktivasyon: tanh/sigmoid blend                   │
│  → PyTorch Lightning + Interval Score Loss                  │
├─────────────────────────────────────────────────────────────┤
│  LAYER 3: BNN Uncertainty (Post-Processing)                 │
│  → MC-Dropout epistemic uncertainty                         │
│  → CAS_final = CAS * (1 - uncertainty_factor)               │
├─────────────────────────────────────────────────────────────┤
│  INFRA: Ray/MLflow (Training) + Triton (Serving)            │
└─────────────────────────────────────────────────────────────┘
```

**Kaynak:** [8-implementasyon-poc-altyapi.md]

## 5.4 Handover Protokolü (Pre-Match → Live)

```python
def handover_pre_to_live(match_id):
    # 1. Pre-Match DQN Çıktısı
    pre_output = dqn.predict(X_static)
    
    # 2. Redis Atomic Transfer (WATCH-MULTI)
    pipe = redis.pipeline(transaction=True)
    payload = {
        "q_pre": pre_output.q_values.tolist(),
        "c0": dqn.hidden_state.numpy().tobytes(),  # LSTM init
        "portfolio": {"exposure": 100.0, "entry_odds": 1.85},
        "ver": atomic_increment(f"match:{match_id}:handover_ver")
    }
    pipe.hset(f"match:{match_id}:handover", mapping=payload)
    pipe.expire(f"match:{match_id}:handover", 5400)  # 90dk TTL
    pipe.execute()
    
    # 3. Teacher Forcing (İlk 10dk)
    for t in range(600):  # 10dk * 1s
        live_agent.step(
            X_live=X_stream[t],
            c0=load_hidden(live_state["c0"]),
            teacher=decode_q_pre(live_state["q_pre"]),
            mode="teacher_forcing"
        )
    
    # 4. Autonomous mode
    live_agent.mode = "autonomous"
```

**Kaynak:** [3-project-oracle-twin-engine.md]

---

# 6. ADAPTİF MEKANİZMALAR

## 6.1 VSNR (Varyans Duyarlı Sinyal-Gürültü Oranı)

**Amaç:** Kaotik anlarda (yüksek varyans) tetikleme eşiğini yükselterek aşırı tepkiyi önlemek.

```python
VSNR = |ΔProb| / sqrt(Var(Last_N_Events))

# Aralık
VSNR ∈ [1.5, 3.5]
VSNR_min_trigger = 1.3  # Gürültü eşiği
VSNR_max_saturation = 4.0  # Sinyal doygunluğu
```

**Kaynak:** [6-otonom-bahis-ai-sistemi.md]

## 6.2 Zaman-Etki Sönümlemesi (Decay Function)

**Kritik Karar:** 85. dakika kırılma noktası (Opta verilerine göre gollerin %12'si 85+ dakikada)

```python
Decay(t) = 1 / (1 + e^{α(t - 85)})

# Kesinleşen Parametreler:
# t_break: 85. dakika
# α: 0.70 (Backtest: Brier -3.2%, MDD -9%, tail-Sortino +0.11)
```

**Gerekçe:** α=0.70, 87. dakikada momentum etkisini %20'ye indirerek gürültü sızıntısı ve sinyal kaybı arasındaki optimum noktayı sağlar.

**Kaynak:** [6-otonom-bahis-ai-sistemi.md]

## 6.3 Adaptif Varyans Koridoru

```python
Corridor_Width = σ_VSNR × √(Liq / Depth_ref)

# Sınırlar
Corridor_Min = 0.6
Corridor_Max = 2.5
Corridor_Width_init ∈ [0.8, 1.8]
```

**Kritik Özellik:** Düşük likidite → Koridor genişler → Öğrenme frenlenir

**Kaynak:** [6-otonom-bahis-ai-sistemi.md]

## 6.4 Sürekli Adaptasyon Skoru (CAS)

```python
CAS = (VSNR × Decay(t) × Confidence_Weight) / Corridor_Liq

# Karar Mekanizması:
if CAS > 1.0:
    trigger_micro_cycle()  # Value Betting / Oran Avcılığı
elif CAS ∈ [0.8, 1.0]:
    prepare_position()  # Pre-Action
else:
    maintain_weights()  # Statik kal

# Öğrenme Oranı Ölçekleme:
η_final = η_base × Sigmoid(CAS - 1)
```

**Kaynak:** [6-otonom-bahis-ai-sistemi.md]

## 6.5 Güven-Ağırlıklı Adaptasyon (Confidence Weight)

**Amaç:** Piyasa manipülasyonuna (fake momentum) karşı koruma

```python
Momentum_Corr = Corr(Prediction_Drift, Market_Drift)

Confidence_Weight = clip(0.4, 1.0, 
    0.4 + 0.6 × tanh(κ × Momentum_Corr × Vol_Idx × (1 + Depth_Ratio))
)

# Kesinleşen Parametreler:
# κ (Kappa): 1.2 (başlangıç)
# η (Öğrenme Oranı): 0.05 (20 maç half-life)
# Aralık: [0.4, 1.0]
```

**Kritik Özellikler:**
- ✅ Düşük güvenli sinyalleri %60'a kadar baskılar
- ✅ Spoofing algılama %23 artış (Depth_Ratio sayesinde)

**Kaynak:** [6-otonom-bahis-ai-sistemi.md]

## 6.6 Piyasa Duyarlılık Faktörü (γ - Gamma)

```python
# Mikro-bahislerin getirdiği Sharpe değişimini izle
γ = ΔSharpe_Ratio(mikro_bahisler)

# Mod Seçimi (Histerezis ile):
γ < -0.08  → Eşgüdüm Modu (histerezis: γ > -0.05)
γ > 0.52   → Liderlik Modu (histerezis: γ < 0.48)
else       → Nötr Mod
```

### Dinamik Aksiyon Matrisi

| Mod | λ Çarpanı | CW Aralığı | Loss Mix | Spread |
|-----|-----------|------------|----------|--------|
| **Eşgüdüm** | 1.15x | [0.4, 1.0] | (0.3, 0.7) | 30% |
| **Liderlik** | 1.40x × (1+√ρ) | [0.7, 1.0] | (0.8, 0.2) | 50% |
| **Nötr** | 1.0x | [0.5, 1.0] | (0.5, 0.5) | 20% |

**Kaynak:** [7-piyasa-sinerjisi-meta-ogrenme.md]

## 6.7 Meta-Öğrenme ve Rejim Geçişleri

### Hibrit Optimizasyon

```python
# Bayesian Optimization (Hız için)
BO_continuous()

# Evrimsel Mutasyon/Crossover (Çeşitlilik için)
# Her 50 epoch'ta tetiklenir
if epoch % 50 == 0:
    evolutionary_mutation()
    evolutionary_crossover()
```

### Knowledge Distillation (Rejim Geçişi)

```python
# Eski Rejim = Teacher, Yeni Rejim = Student
L_total = w(t) × L_student + (1 - w(t)) × L_teacher

# Sigmoidal Geçiş (0.3 → 1.0 / 40-60 maç)
w(t) = 0.3 + 0.7 × sigmoid(t - T/2)

# p_BCD > 0.9 ise hızlandır (30 maç)
T = 30 if p_BCD > 0.9 else 60
```

**Kaynak:** [7-piyasa-sinerjisi-meta-ogrenme.md]

---

# 7. RİSK VE STRATEJİ KATMANI

## 7.1 CVaR-Constrained Thompson Sampling

```python
def constrained_thompson_sampling(priors, cvar_limit=0.05, bankroll=1000):
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
        bankroll * 0.05,                       # Max %5 single bet
        bankroll * samples[best_action] * 0.3  # %30 fractional
    )
    
    return best_action, stake
```

**Kaynak:** [4-canlı-futbol-simülasyon-sistemi.md], [9-bigplan-manifestosu.md]

## 7.2 Sabit Risk Limitleri

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

**Kaynak:** [9-bigplan-manifestosu.md]

## 7.3 Ödül Fonksiyonu (Reward Function)

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

**Kaynak:** [1-bahis-tahmin-platformu.md], [9-bigplan-manifestosu.md]

## 7.4 Portföy Korelasyonu Yönetimi

```python
# Korelasyon Matrisi
R = Correlation_Matrix(bahisler)

# Etkin Bahis Sayısı (Diversification Measure)
N_eff = 1 / (w.T @ R @ w)

# Dinamik Lambda (Kelly Ayarlaması)
λ = base_lambda × mode_mult × (1 + √avg_corr)

# Öğrenme Hızı Sönümlemesi
eta_effective = base_eta × min(1, N_eff / K)

# Liderlik Modunda (γ > 0.52) ve ρ_avg > 0.3:
eta = base_eta / (1 + ρ_avg × 2)
```

**Kaynak:** [7-piyasa-sinerjisi-meta-ogrenme.md]

---

# 8. ÜRETİM ALTYAPISI

## 8.1 Docker Stack

```yaml
services:
  # Veri Katmanı
  clickhouse:
    image: clickhouse/clickhouse-server:latest
    # 1M/s ingestion
  
  timescale:
    image: timescale/timescaledb:latest-ha
    # OLTP + Continuous Aggregates
  
  redis:
    image: redis:7.2
    command: ["--maxmemory", "4gb"]
    # Cache + Online Store
  
  neo4j:
    image: neo4j:5
    # Knowledge Graph
  
  milvus:
    image: milvusdb/milvus:latest
    # Vector Store
  
  # Stream Processing
  kafka:
    image: confluentinc/cp-kafka:latest
  
  flink:
    image: flink:1.17
    # CDC Processing
  
  # ML Pipeline
  ray-head:
    image: rayproject/ray:2.9.0-py310
    # Distributed Training
  
  mlflow:
    image: mlflow/mlflow:2.7.1
    # Model Registry
  
  triton:
    image: nvcr.io/nvidia/tritonserver:24.01-py3
    # Model Serving (FP16)
  
  # Monitoring
  prometheus:
    image: prom/prometheus
  
  grafana:
    image: grafana/grafana
```

**Kaynak:** [8-implementasyon-poc-altyapi.md], [9-bigplan-manifestosu.md]

## 8.2 Kubernetes Deployment

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

# Canary Deployment
spec:
  canaryTrafficPercent: 10  # %10 yeni modele
```

**Kaynak:** [9-bigplan-manifestosu.md]

## 8.3 Circuit Breaker Matrisi

| Bileşen | Threshold | Timeout | Fallback | Kaynak |
|---------|-----------|---------|----------|--------|
| DataPlant | 3 failures | 30s | CanonicalMatch (stale OK) | [9-bigplan] |
| IntelligencePlant | 2 failures | 10s | Student → Redis → Rule-Based → Skip | [9-bigplan] |
| FeatureStore | 5 failures | 5s | Computed on-the-fly | [9-bigplan] |
| StateStore (Redis) | 3 failures | 15s | Caffeine LRU (1000 match) | [9-bigplan] |

## 8.4 Graceful Degradation (4-Tier Fallback)

```python
def graceful_degrade(match_id):
    # 1. Student Model (GraphSAGE)
    try:
        pred = student_model.predict(match_id, timeout=5)
        if pred.uncertainty < 0.3:
            return pred
    except: pass
    
    # 2. Redis Cache
    try:
        cached = redis.get(f"last_prediction:{match_id}")
        if cached and time.time() - cached.ts < 300:
            return cached
    except: pass
    
    # 3. Rule-Based Model
    try:
        return baseline_stats_model.predict(match_id)
    except: pass
    
    # 4. Skip (Tüm sistemler fail)
    return {"action": "SKIP", "reason": "all_systems_down"}
```

**Kaynak:** [9-bigplan-manifestosu.md]

## 8.5 VRAM ve Performans Optimizasyonu

| Teknik | Kazanım | Trade-off | Kaynak |
|--------|---------|-----------|--------|
| FSDP + CPU-offload | %35-45 VRAM azalma | CPU overhead | [8-impl] |
| Activation Checkpointing | %40 VRAM tasarruf | %30 compute artış | [8-impl] |
| TensorRT FP16 | +%40 throughput, 2x memory compress | Precision | [8-impl] |
| Gradient Accumulation | Effective batch size × N | Training time | [8-impl] |
| A100 MIG | 3g.20gb train + 1g.5gb serve | Partition overhead | [8-impl] |

---

# 9. UYGULAMA YOL HARİTASI

## 9.1 Curriculum Learning Phases

```
PHASE 1: Prematch Only (Basit)
├── Target: Win rate > 55%
├── Model: DQN on static features
├── Risk: VaR limitleri aktif
└── Süre: 4-6 hafta

    ↓

PHASE 2: Live Only (Orta)
├── Target: Win rate > 52%
├── Model: LSTM + PPO
├── Risk: CVaR-Thompson aktif
└── Süre: 6-8 hafta

    ↓

PHASE 3: Combined (Zor)
├── Target: Win rate > 50%
├── Model: HRL (Manager + Workers)
├── Risk: Fractional Kelly (0.75)
└── Süre: 8-12 hafta

    ↓

PHASE 4: Full System (Çok Zor)
├── Target: Sharpe > 0.8
├── Model: Full HRL + Meta-Learning
├── Risk: All mechanisms active
└── Süre: Sürekli evrim
```

**Kaynak:** [9-bigplan-manifestosu.md]

## 9.2 PoC Altyapı Checklist

### Aşama 1: Veri Katmanı (Hafta 1-2)
- [ ] ClickHouse kurulumu + schema
- [ ] TimescaleDB kurulumu + hypertable
- [ ] Kafka topics oluşturma
- [ ] API-Football v3 adapter yazımı
- [ ] Redis cache kurulumu
- [ ] Flink CDC pipeline

### Aşama 2: Dijital İkiz (Hafta 3-4)
- [ ] Neo4j Knowledge Graph
- [ ] Milvus Vector Store
- [ ] GNN model eğitimi
- [ ] Monte Carlo simülasyon modülü

### Aşama 3: Zeka Katmanı (Hafta 5-8)
- [ ] Graph-LSTM Encoder
- [ ] LSTM-State-Space Core
- [ ] TFT Decoder
- [ ] HRL Manager Agent
- [ ] Worker Agents (Live + Prematch)
- [ ] Feast Feature Store entegrasyonu

### Aşama 4: Risk Katmanı (Hafta 9-10)
- [ ] CVaR-Thompson Sampling
- [ ] Fractional Kelly implementasyonu
- [ ] Risk limitleri modülü
- [ ] Portföy korelasyonu yönetimi

### Aşama 5: Production (Hafta 11-12)
- [ ] Docker Compose stack
- [ ] Kubernetes Helm charts
- [ ] KServe InferenceService
- [ ] Triton FP16 optimization
- [ ] Prometheus + Grafana monitoring
- [ ] Circuit Breaker entegrasyonu

### Aşama 6: Adaptif Mekanizmalar (Hafta 13-16)
- [ ] VSNR implementasyonu
- [ ] Decay function
- [ ] CAS formula
- [ ] Confidence Weight
- [ ] γ (Gamma) Market Sensitivity
- [ ] Meta-Learning döngüsü

---

# 10. KAYNAK ETİKETLEME MATRİSİ

## Teknoloji → Kaynak Eşleştirmesi

| Teknoloji/Konsept | Kaynak Dosya |
|-------------------|--------------|
| **ClickHouse (1M/s)** | [5-rdql-sanal-betting-sistemi.md] |
| **TimescaleDB + Hypertable** | [2-production-ready-architecture.md], [3-project-oracle-twin-engine.md] |
| **Twin Database (Hot/Cold)** | [3-project-oracle-twin-engine.md] |
| **Neo4j Knowledge Graph** | [4-canlı-futbol-simülasyon-sistemi.md], [9-bigplan-manifestosu.md] |
| **Milvus Vector Store** | [4-canlı-futbol-simülasyon-sistemi.md] |
| **Feast Feature Store** | [5-rdql-sanal-betting-sistemi.md], [9-bigplan-manifestosu.md] |
| **HRL (UCB + PPO/DQN)** | [1-bahis-tahmin-platformu.md], [9-bigplan-manifestosu.md] |
| **Graph-LSTM + TFT** | [5-rdql-sanal-betting-sistemi.md] |
| **LSTM-State-Space** | [5-rdql-sanal-betting-sistemi.md] |
| **GNN + Monte Carlo** | [4-canlı-futbol-simülasyon-sistemi.md] |
| **Bidirectional Cross-Attention** | [3-project-oracle-twin-engine.md] |
| **MaskedTensor** | [2-production-ready-architecture.md] |
| **CVaR-Thompson Sampling** | [4-canlı-futbol-simülasyon-sistemi.md] |
| **Fractional Kelly** | [5-rdql-sanal-betting-sistemi.md], [9-bigplan-manifestosu.md] |
| **VSNR** | [6-otonom-bahis-ai-sistemi.md] |
| **Decay (85dk)** | [6-otonom-bahis-ai-sistemi.md] |
| **CAS Formula** | [6-otonom-bahis-ai-sistemi.md] |
| **Confidence Weight** | [6-otonom-bahis-ai-sistemi.md] |
| **γ (Gamma) Market Sensitivity** | [7-piyasa-sinerjisi-meta-ogrenme.md] |
| **Knowledge Distillation** | [7-piyasa-sinerjisi-meta-ogrenme.md], [9-bigplan-manifestosu.md] |
| **3-Layer Architecture** | [8-implementasyon-poc-altyapi.md] |
| **LightGBM-Quantile** | [8-implementasyon-poc-altyapi.md] |
| **HyperNetworks** | [8-implementasyon-poc-altyapi.md] |
| **BNN Uncertainty (MC-Dropout)** | [8-implementasyon-poc-altyapi.md], [9-bigplan-manifestosu.md] |
| **MTGP** | [8-implementasyon-poc-altyapi.md] |
| **FSDP + CPU-offload** | [8-implementasyon-poc-altyapi.md] |
| **Triton FP16** | [8-implementasyon-poc-altyapi.md] |
| **Circuit Breaker Matrix** | [9-bigplan-manifestosu.md] |
| **Graceful Degradation** | [9-bigplan-manifestosu.md] |
| **CloudEvents Schema** | [9-bigplan-manifestosu.md] |
| **RDP Compression** | [3-project-oracle-twin-engine.md] |
| **Protobuf (TwinDelta)** | [3-project-oracle-twin-engine.md] |
| **Emergency Hedge API** | [3-project-oracle-twin-engine.md] |
| **Handover Protocol** | [3-project-oracle-twin-engine.md] |
| **Teacher Forcing** | [3-project-oracle-twin-engine.md] |
| **Ray.io Distributed** | [5-rdql-sanal-betting-sistemi.md] |
| **PettingZoo Environment** | [5-rdql-sanal-betting-sistemi.md] |

---

# 📊 NİHAİ PERFORMANS HEDEFLERİ

## Sistem Metrikleri

| Metrik | Hedef | Strateji |
|--------|-------|----------|
| **Latency (p99)** | < 60ms | Triton FP16 + Priority Queue |
| **Throughput** | +40% | TensorRT optimization |
| **VRAM (Serving)** | 16Gi | FSDP + CPU offload |
| **VRAM (Training)** | 32Gi | Activation checkpointing |
| **Freshness SLO** | > 95% | Auto-skip stale data |
| **Fallback Rate** | < 10% | 4-tier degradation ladder |
| **Test Coverage** | > 80% | Pytest mandatory |

## ROI Hedefleri

| Aşama | ROI Hedefi | Risk Kontrolü |
|-------|-----------|---------------|
| Phase 1 (Prematch) | Win rate > 55% | VaR limitleri |
| Phase 2 (Live) | Win rate > 52% | CVaR-Thompson |
| Phase 3 (Combined) | Win rate > 50% | Fractional Kelly (0.75) |
| Phase 4 (Full HRL) | Sharpe > 0.8 | All mechanisms |

---

# 🎯 SONUÇ

## Sistem Karakteristikleri

1. **Süper-Rasyonel:** İnsanın analitik kapasitesini aşan, duygu içermeyen karar mekanizması
2. **Sürekli Yaşayan:** Meta-öğrenme ile adapte olan, rejim geçişlerini yöneten dijital varlık
3. **Modüler:** Plant-based architecture (yeni özellik = yeni tesis)
4. **Resilient:** 6 bileşende Circuit Breaker, 4-tier fallback ladder
5. **Risk-Aware:** CVaR-Thompson + Fractional Kelly + VSNR + CAS
6. **Observable:** Prometheus + Grafana + PagerDuty + SLO dashboard

## Kritik Başarı Faktörleri

- ✅ Twin Database (ClickHouse + TimescaleDB)
- ✅ Dijital İkiz (Neo4j + Milvus + GNN)
- ✅ HRL (UCB Manager + LSTM/DQN Workers)
- ✅ Entegre Model (Graph-LSTM + State-Space + TFT)
- ✅ Adaptif Mekanizmalar (VSNR, CAS, Decay, γ)
- ✅ Risk Yönetimi (CVaR-Thompson + Kelly)
- ✅ Meta-Öğrenme (BO + Evolutionary + KD)
- ✅ Graceful Degradation (Circuit Breaker + 4-tier)

---

**Doküman Versiyonu:** v1.0  
**Oluşturma Tarihi:** 03.01.2026  
**Kaynak:** 9 münazara özet dosyası (toplam 4573 satır)  
**Model:** Claude 4.5 Opus  
**Agent:** Antigravity  
