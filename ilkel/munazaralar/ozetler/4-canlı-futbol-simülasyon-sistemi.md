# Canlı Futbol Simülasyon & Tahmin Sistemi - İnsan Taklidi AI

## 🎯 Proje Vizyonu
**"Futbol verilerini API ile çekip, canlı yaşayan ve insanı taklit eden, zihninde simülasyon yapan, sürekli öğrenen bir tahmin sistemi"**

Soğuk veri, pre-match verileri, canlı maç verileri, istatistikler ve bahis oranlarını emen; strateji geliştirip sürekli öğrenen; tahminlerini mükemmelleştiren; tekli/çoklu/sistemli kuponlar oluşturabilen; maçları oynanmadan zihninde simüle eden ve canlı maç esnasında gerçek verilerle öğrenebilen bir sistem.

## 🏗️ 1. Temel Mimari Kararları

### Event-Driven Microservices Mimarisi
- **Veri Akışı:** Kafka ile event-driven mimari
- **Deployment:** Kubernetes
- **ML Pipeline:** Airflow orchestration
- **Stream Processing:** Apache Flink (Stateful Functions)
- **Latency Hedefi:** Sub-second (<100ms canlı maç için)

### Kritik Teknoloji Seçimleri
```
API Ingestion → Kafka Topics (prematch/canlı/istatistik) → Flink → ML Servisleri
```

## 🧠 2. GNN Tabanlı Dijital İkiz Simülasyon Motoru

### Graph Neural Networks (GNN) Mimarisi
**Amaç:** Maçları oynanmadan "zihninde canlandırma"

**Grafik Yapısı:**
- **Düğümler (Nodes):** Takımlar ve Oyuncular
- **Kenarlar (Edges):** Pas trafiği, formasyonlar, oyun planı
- **Özellikler (Features):** Oyuncu form vektörleri, takım gücü, sakatlıklar

### Monte Carlo Simülasyonu + Bayesian Update

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

## 🤖 3. Reinforcement Learning Ajanı (CVaR-Kısıtlı Thompson Sampling)

### Çatışma Çözümü: Kelly+CVaR vs Thompson Sampling
**Karar:** CVaR ile kısıtlı Thompson Sampling Bandit

**Gerekçe:**
- ✅ Thompson Sampling → Non-stationary ortama hızlı adaptasyon
- ✅ CVaR kısıtı → Risk yönetimi (%5 minimum güvenlik marjı)
- ✅ Her 5 dakikada beta dağılımı güncelleme

### CVaR-Kısıtlı Thompson Sampling Implementasyonu

```python
def constrained_thompson_sampling(priors, cvar_limit=0.05):
    # Beta dağılımından örnekleme
    samples = [beta.rvs(alpha, beta) for alpha, beta in priors]
    
    # CVaR filtresi (Risk yönetimi)
    valid_actions = [
        i for i in range(len(samples)) 
        if np.percentile(samples, 95) >= cvar_limit
    ]
    
    # En iyi aksiyonu seç
    return np.argmax([samples[i] for i in valid_actions])
```

### RL Ajanı Mimarisi

```python
class BettingAgent:
    def __init__(self):
        self.model = DQN()  # Deep Q-Network
        self.thompson = ThompsonSampling()
        
    def act(self, state):
        # State: GNN Embedding + Canlı Oran + Sentiment
        state_vector = torch.cat([
            gnn_embedding,
            live_odds,
            sentiment_score
        ])
        
        # CVaR kısıtlı aksiyon seçimi
        action = self.thompson.select_action(
            state_vector,
            cvar_limit=0.05
        )
        
        return action  # Kupon oluşturma kararı
```

**Reward Fonksiyonu:** ROI + Risk/Likidite kısıtları + Portföy optimizasyonu

## 📊 4. Knowledge Graph & Vector Store Entegrasyonu

### Kritik Tespit
GNN simülasyonu, gerçekçi priors üretmek için zengin bilgi tabanına ihtiyaç duyar.

### Teknoloji Kararları
- **Knowledge Graph:** Neo4j (Takım formasyonları, sakatlıklar, tarihli maçlar)
- **Vector Store:** Milvus/Pinecone (Oyuncu özellikleri, performans vektörleri)

### Veri Akışı ve Senkronizasyon Mekanizması

**Sorun:** Knowledge Graph ve Vector Store'dan GNN'e gerçek zamanlı veri akışı nasıl sağlanır?

**Çözüm:** CDC + Kafka + Flink + Transactional Outbox Pattern

```
┌─────────────────────────────────────────────────────────┐
│ Knowledge Graph (Neo4j)                                 │
│ - Takım formasyonları, sakatlıklar, tarihli maçlar     │
└─────────────────────────────────────────────────────────┘
                    ↓ (CDC - Debezium)
┌─────────────────────────────────────────────────────────┐
│ Kafka Topics                                            │
│ - graph_events, player_updates, match_events           │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Flink Stream Processing                                 │
│ - Adjacency Matrix güncelleme (dinamik grafik)         │
│ - Event-time watermark (exactly-once garantisi)        │
│ - Stateful Functions (RocksDB state)                   │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ Vector Store (Milvus)                                   │
│ - Oyuncu form vektörleri                               │
│ - Async fetch + Redis cache (TTL 30s)                  │
│ - Vector versioning (atomic swap)                      │
└─────────────────────────────────────────────────────────┘
                    ↓ (On-the-fly injection)
┌─────────────────────────────────────────────────────────┐
│ GNN Inference (PyTorch Geometric)                       │
│ - Incremental Neighbor Sampling                        │
│ - Hybrid Embedding (graph + vector)                    │
└─────────────────────────────────────────────────────────┘
```

### Tutarlılık Garantileri

**1. Transactional Outbox Pattern (Neo4j)**
```
Neo4j Trigger → Outbox Table → Kafka Transaction → Debezium → Flink
```

**2. Vector Versioning (Milvus)**
```python
# Atomic swap ile tutarlılık
milvus.upsert(
    collection="player_embeddings",
    data=new_vectors,
    version=atomic_increment("vector_version")
)
```

**3. Flink Exactly-Once Garantisi**
- Event-time processing + Watermark
- Late-event side-output
- Idempotent event processing (event_id ile)

### Incremental Neighbor Sampling (Sub-Second Latency)

```python
# Tüm grafiği yeniden yüklemeden canlı veri işleme
class IncrementalGNN:
    def process_event(self, event):
        # Flink state'inden güncel komşuları çek
        event_nodes = flink_state.get_affected_nodes(event)
        
        # PyG NeighborLoader ile subgraph oluştur
        subgraph = NeighborLoader(
            data=graph_data,
            num_neighbors=[-1],
            input_nodes=event_nodes
        )
        
        # GNN inference (sadece etkilenen düğümler)
        embedding = gnn_model(subgraph)
        
        return embedding
```

**Performans:** <200ms latency (10 saniyelik adaptive batching ile)

## 🎭 5. İnsan Taklidi: Contextual Bias Layer (NLP + Sentiment)

### BERT Tabanlı Duygu Analizi
**Amaç:** Sosyal medya ve haber akışını işleyerek "insan gibi" önyargı oluşturma

**Veri Kaynakları:**
- Sosyal medya (Twitter, Reddit)
- Haber siteleri
- Uzman yorumları

### NLP Pipeline

```python
class SentimentAnalyzer:
    def __init__(self):
        self.bert = BERTModel.from_pretrained("bert-base-uncased")
        
    def analyze(self, text):
        # Tokenization + BERT inference
        tokens = self.tokenizer(text)
        sentiment = self.bert(tokens)
        
        return sentiment  # [-1, 1] arası skor
```

### Contextual Bias Layer (GNN'ye Enjeksiyon)

```python
# Sentiment skorunu GNN simülasyonuna prior olarak ekle
def inject_contextual_bias(gnn_priors, sentiment_score):
    # Sentiment → Olasılık ağırlığı
    bias_weight = sigmoid(sentiment_score)
    
    # GNN priors'ı ağırlıklandır
    biased_priors = gnn_priors * (1 + 0.2 * bias_weight)
    
    return biased_priors
```

**Kritik Karar:** Sentiment, GNN simülasyonuna %20'ye kadar etki edebilir (aşırı önyargıyı önlemek için)

## 🔄 6. State Assembler: GNN → RL Ajanı Entegrasyonu

### Sorun
RL Ajanı, sadece GNN simülasyonunu değil, piyasa verilerini ve NLP önyargısını da görmeli.

### Çözüm: Flink State-Based Assembler

**Feast Feature Store vs Flink RocksDB State**
- ❌ Feast → Ekstra latency (network hop)
- ✅ Flink RocksDB State → Düşük latency (<50ms)

```
┌─────────────────────────────────────────────────────────┐
│ Flink State (RocksDB)                                   │
│ ← Neo4j CDC + Milvus Embeddings + Live Odds + NLP      │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ GNN Inference (PyTorch Geometric)                       │
│ → Graph Embedding                                       │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ State Assembler (Tensör Birleştirme)                   │
│ state = [gnn_emb, live_odds, sentiment, market_depth]  │
└─────────────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────┐
│ RL Agent (CVaR-Constrained Thompson Sampling)          │
│ → Kupon oluşturma kararı                               │
└─────────────────────────────────────────────────────────┘
```

### Adaptive Batching (Flink)

```python
# Her 10 saniyelik window için
def adaptive_batching(events):
    # 1. Düğüm olayları → Batch accumulation
    batch = accumulate_events(events, window="10s")
    
    # 2. Async Milvus fetch → Preload embeddings
    embeddings = async_fetch_embeddings(batch.player_ids)
    
    # 3. PyG NeighborLoader → Mini-batch subgraph
    subgraph = create_subgraph(batch, embeddings)
    
    # 4. GNN inference → Simülasyon
    simulation = gnn_inference(subgraph)
    
    return simulation
```

## 🎯 7. Kritik Teknik Kararlar

### 1. Veri Akışı ve Senkronizasyon
- ✅ Neo4j CDC (Debezium) → Kafka → Flink
- ✅ Transactional Outbox Pattern (tutarlılık)
- ✅ Vector Versioning (Milvus atomic swap)
- ✅ Exactly-Once garantisi (Flink watermark)

### 2. GNN Besleme Stratejisi
- ✅ Incremental Neighbor Sampling (tüm grafiği yeniden yüklemeden)
- ✅ On-the-fly embedding injection (Milvus → GNN)
- ✅ Dinamik Adjacency Matrix (Flink stateful update)

### 3. Risk Yönetimi
- ❌ Sadece ROI odaklı reward
- ✅ CVaR-kısıtlı Thompson Sampling (risk + adaptasyon)
- ✅ Her 5 dakikada beta dağılımı güncelleme

### 4. State Management
- ❌ Feast Feature Store (ekstra latency)
- ✅ Flink RocksDB State (<50ms latency)

### 5. İnsan Taklidi
- ✅ BERT tabanlı sentiment analizi
- ✅ Contextual Bias Layer (%20 maksimum etki)
- ✅ Sosyal medya + haber akışı entegrasyonu

## 📐 8. Eksiksiz Mimari Tree Graph

```
📦 FOOTBALL PREDICTION SYSTEM
│
├── 🌐 DATA INGESTION LAYER
│   ├── API Gateway (REST/WebSocket)
│   ├── Kafka Topics
│   │   ├── prematch_data
│   │   ├── live_match_data
│   │   ├── betting_odds
│   │   ├── graph_events
│   │   └── sentiment_stream
│   └── Schema Registry
│
├── 💾 STORAGE LAYER
│   ├── Knowledge Graph (Neo4j)
│   │   ├── Teams, Players, Matches
│   │   ├── Formations, Injuries
│   │   └── CDC (Debezium) → Kafka
│   ├── Vector Store (Milvus)
│   │   ├── Player Embeddings
│   │   ├── Team Form Vectors
│   │   └── Vector Versioning
│   └── Time-Series DB (InfluxDB)
│       └── Live Match Stats
│
├── 🔄 STREAM PROCESSING LAYER (Flink)
│   ├── Stateful Functions (RocksDB)
│   ├── Event-Time Watermark
│   ├── Exactly-Once Semantics
│   ├── Adjacency Matrix Updater
│   └── Adaptive Batching (10s window)
│
├── 🧠 AI/ML LAYER
│   ├── GNN Simulation Engine (PyTorch Geometric)
│   │   ├── Incremental Neighbor Sampling
│   │   ├── Monte Carlo Simulation (10k iterations)
│   │   └── Bayesian Update (live)
│   ├── NLP Sentiment Analyzer (BERT)
│   │   ├── Social Media Stream
│   │   ├── News Feed
│   │   └── Contextual Bias Layer
│   └── RL Agent (DQN + Thompson Sampling)
│       ├── CVaR Constraint (5%)
│       ├── State Assembler
│       └── Kupon Generator
│
├── 🎯 DECISION LAYER
│   ├── Portföy Optimizasyonu (Kelly+CVaR)
│   ├── Stake Allocator
│   └── Kupon Oluşturucu (Tekli/Çoklu/Sistem)
│
├── 📊 MONITORING & FEEDBACK
│   ├── Prometheus Metrics
│   ├── Grafana Dashboard
│   ├── Model Performance Tracker
│   └── Backtesting Engine
│
└── 🚀 DEPLOYMENT (Kubernetes)
    ├── Helm Charts
    ├── KServe (Model Serving)
    ├── Canary Deployment (A/B Test)
    └── Airflow (ML Pipeline Orchestration)
```

## 🚀 Deployment ve Operasyon

### Kubernetes Deployment
- **Model Serving:** KServe
- **A/B Testing:** Canary deployment
- **Orchestration:** Airflow
- **Monitoring:** Prometheus + Grafana

### Performans Hedefleri
- **Canlı Maç Latency:** <100ms
- **GNN Inference:** <200ms
- **State Assembly:** <50ms
- **End-to-End:** <500ms

### Backtesting
- Geriye dönük simülasyon
- Günlük/Haftalık/Aylık metrik dashboard
- Latency, margin, cashout, limitler analizi

## 🎓 Sonuç

Bu sistem, futbol verilerini API ile çekerek:
1. ✅ **Zihninde simülasyon yapar** (GNN + Monte Carlo)
2. ✅ **Canlı veriyle öğrenir** (Bayesian Update)
3. ✅ **İnsan gibi düşünür** (NLP Sentiment + Contextual Bias)
4. ✅ **Risk yönetir** (CVaR-kısıtlı Thompson Sampling)
5. ✅ **Sürekli gelişir** (RL Agent + Feedback Loop)
6. ✅ **Gerçek zamanlı çalışır** (Sub-second latency)

**Kritik Başarı Faktörleri:**
- Event-driven mimari (Kafka + Flink)
- GNN tabanlı dijital ikiz
- CVaR-kısıtlı Thompson Sampling
- Incremental Neighbor Sampling
- Transactional Outbox Pattern
- Flink RocksDB State (düşük latency)

