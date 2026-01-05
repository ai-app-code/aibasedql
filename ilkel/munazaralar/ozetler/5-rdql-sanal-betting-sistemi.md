# RDQL Sanal Betting Sistemi - Öğrenen & Gelişen AI

## 🎯 Proje Vizyonu
**"API ile beslenen, sürekli öğrenen, kupon stratejileri geliştiren, sanal dünyada kazanç elde eden AI sistemi"**

Pre-match ve live-match verilerini API'den çekerek; takım, lig, oyuncu, formasyon ve oyun gücü konularında öğrenen ve gelişen; kupon stratejileri geliştirerek sanal ortamda (paper trading) kazanç elde eden bir Recurrent Deep Q-Learning (RDQL) sistemi.

## 🏗️ 1. Veri Mimarisi (3-Tier Storage)

### Kritik Tespit: Canlı Veri Yönetimi Sorunu
**Sorun:** Canlı maçlarda bazı veriler kümülatif (korner), bazıları dalgalı (hakimiyet), oranlar sürekli değişir, bahis türleri açılır/kapanır.

**Çözüm:** Hot-Warm-Cold 3-Tier Mimari

### Teknoloji Kararları (5 Tur Münazara Sonucu)

**Ana Depolama: ClickHouse** (TimescaleDB yerine)
- ✅ 1M/s tick ingestion (TimescaleDB'den 5x hızlı)
- ✅ Vectorized sorgular (PB scale historical data)
- ✅ ReplacingMergeTree (idempotent odds upsert)
- ✅ Materialized Views (gerçek zamanlı hakimiyet rollup)
- ✅ TTL ile S3'e soğuk taşıma

```sql
-- ClickHouse Tablo Yapısı
CREATE TABLE live_odds (
    time DateTime64(3),
    match_id UInt32,
    market LowCardinality(String),
    odds Float64,
    version UInt64
) ENGINE = ReplacingMergeTree(version)
ORDER BY (time, match_id, market)
PARTITION BY toYYYYMMDD(time)
TTL time + INTERVAL 30 DAY TO DISK 's3_cold';

-- Materialized View (Hakimiyet Rollup)
CREATE MATERIALIZED VIEW dominance_1s_mv
ENGINE = AggregatingMergeTree()
ORDER BY (match_id, time_bucket)
AS SELECT
    match_id,
    toStartOfSecond(time) AS time_bucket,
    avgState(dominance) AS avg_dominance,
    maxState(dominance) AS max_dominance
FROM live_stats
GROUP BY match_id, time_bucket;
```

**Feature Store: Feast** (Online + Offline)
- **Online Store:** Redis (sub-millisecond latency)
- **Offline Store:** Delta Lake (Hudi MOR format)

### CDC Pipeline: ClickHouse → Feast

**Sorun:** ClickHouse native CDC desteklemez.

**Çözüm:** Kafka Engine + Materialized View

```sql
-- Kafka Engine Tablosu
CREATE TABLE feast_cdc (
    match_id String,
    feature_name String,
    value Float64,
    event_ts DateTime64(3)
) ENGINE = Kafka
SETTINGS kafka_broker_list = 'kafka:9092',
         kafka_topic_list = 'feast_features',
         kafka_format = 'JSONEachRow';

-- Materialized View (Latest State Push)
CREATE MATERIALIZED VIEW feast_cdc_mv TO feast_cdc AS
SELECT
    match_id,
    feature_name,
    argMax(value, event_ts) AS value,  -- Latest value
    max(event_ts) AS event_ts
FROM live_features FINAL
GROUP BY match_id, feature_name;
```

**Flink CDC Pipeline:**
```
ClickHouse Kafka Engine → Flink (exactly-once) → Feast
                                ↓
                    Redis (Lua CAS version check)
                    Delta Lake (Hudi MOR upsert)
```

### Hudi Merge-on-Read (MOR) Stratejisi

**Çatışma:** Copy-on-Write vs Merge-on-Read

**Karar:** Merge-on-Read (MOR) - Yüksek frekanslı güncellemeler için

**Gerekçe:**
- ✅ Log-append yazma (write amplification %60 azaltma)
- ✅ Async compaction (5-10 dakika)
- ✅ 128MB log file size
- ✅ Hourly clustering

```yaml
# Hudi Table Configuration
hudi_table:
  type: MERGE_ON_READ
  recordkey.field: "match_id,feature"
  precombine.field: "event_ts"
  partition.path.field: "match_date"
  hoodie.compaction.async.enabled: true
  hoodie.compaction.async.wait.timeout: 300
  hoodie.compaction.strategy: BINLOG
  hoodie.compaction.max.delta: 10
  hoodie.log.file.size: 128MB
  hoodie.small.file.limit: 100MB
```

## 🧠 2. Hibrit Model Mimarisi (Graph-LSTM + TFT)

### Çatışma Çözümü: Kalman vs LSTM-State-Space
**Karar:** LSTM-State-Space (Kalman'ın lineer varsayımı futbola uymaz)

### 3-Katmanlı Mimari

**1. Graph-LSTM Encoder (Spatial-Temporal)**
- **Amaç:** Oyuncu/takım ilişkilerini ve zaman serisini birleştirme
- **Teknoloji:** GNN (Graph Neural Networks) + LSTM

```python
class GraphLSTM(nn.Module):
    def __init__(self, node_dim=64, hidden_dim=128):
        super().__init__()
        self.gnn = GraphConv(node_dim, hidden_dim)
        self.lstm = nn.LSTM(hidden_dim, hidden_dim)
        
    def forward(self, x_nodes, edge_index, batch_index):
        # GNN: Spatial embedding (oyuncu etkileşimleri)
        x_graph = self.gnn(x_nodes, edge_index)
        
        # Global Attention Pooling (node → graph vector)
        pooled = global_attention_pool(x_graph, batch_index)
        
        # Sequence formatına dönüştür (Batch, Seq, Embed)
        lstm_input = pooled.view(batch_size, seq_len, -1)
        
        # LSTM: Temporal dynamics
        output, (h, c) = self.lstm(lstm_input)
        
        return output, h, c
```

**2. LSTM-State-Space Core (Dynamics Modeling)**
- **Amaç:** Oyun dinamiklerini non-lineer modelleme
- **Formül:** $h_t = \sigma(W_h [h_{t-1}, x_t] + b_h)$

```python
class LSTMStateSpace(nn.Module):
    def forward(self, spatial_embedding, market_data):
        # Spatial + Market verilerini birleştir
        x_t = torch.cat([spatial_embedding, market_data], dim=-1)
        
        # LSTM dynamics
        h_t, c_t = self.lstm_cell(x_t, (h_prev, c_prev))
        
        # Residual connection (daha iyi gradient flow)
        h_t = h_t + spatial_embedding
        
        return h_t, c_t
```

**3. TFT Decoder (Variable Selection + Interpretability)**
- **Amaç:** Hangi özelliğin (korner, oran, formasyon) önemli olduğunu belirleme
- **Teknoloji:** Temporal Fusion Transformer

```python
class TemporalFusionTransformer(nn.Module):
    def forward(self, lstm_state, static_features):
        # Variable Selection Network
        # Hangi feature'ın o an önemli olduğunu öğren
        attention_weights = self.variable_selection(
            lstm_state, 
            static_features  # lig, takım gücü, formasyon
        )
        
        # Weighted feature combination
        output = torch.sum(
            attention_weights.unsqueeze(-1) * lstm_state,
            dim=1
        )
        
        return output, attention_weights
```

### Entegre Model

```python
class IntegratedModel(nn.Module):
    def __init__(self):
        super().__init__()
        self.graph_lstm = GraphLSTM()      # Encoder
        self.lstm_core = LSTMStateSpace()  # Core
        self.tft_decoder = TemporalFusionTransformer()  # Decoder
    
    def forward(self, x_player, edge_index, market_data, static_features):
        # 1. Graph-LSTM Encoder (Spatial-Temporal)
        spatial_embedding, h, c = self.graph_lstm(x_player, edge_index)
        
        # 2. LSTM-State-Space Core (Dynamics)
        temporal_state, _ = self.lstm_core(spatial_embedding, market_data)
        
        # 3. TFT Decoder (Variable Selection)
        output, attention_weights = self.tft_decoder(
            temporal_state, 
            static_features
        )
        
        return output, attention_weights  # RL agent'a hem tahmin hem interpretability
```

### Graph-LSTM Veri Formatı

**Sorun:** GNN çıktısı LSTM'e nasıl beslenecek?

**Çözüm:** Global Attention Pooling + Sequence Format

```python
# Her zaman adımı için:
# 1. GNN node embeddings → Global Attention Pooling → graph vector
g_t = global_attention_pool(node_embeddings, batch_index)

# 2. Market verileriyle birleştir
x_t = torch.cat([g_t, market_odds, dominance], dim=-1)

# 3. Sequence formatında LSTM'e besle
lstm_input = x_t.view(batch_size, seq_len, embed_dim)  # (B, T, D)
```

## 🤖 3. RDQL Öğrenme Stratejisi

### Recurrent Deep Q-Learning (RDQL)
**Amaç:** Kupon stratejileri geliştirerek sanal kazanç elde etme

### CVaR-Kısıtlı Reward Fonksiyonu

```python
class RDQLAgent:
    def __init__(self):
        self.model = IntegratedModel()
        self.cvar_limit = 0.05  # %5 risk limiti
        
    def compute_reward(self, action, outcome):
        # ROI hesapla
        roi = (outcome.payout - action.stake) / action.stake
        
        # CVaR kısıtı (Risk yönetimi)
        portfolio_risk = self.compute_cvar(self.portfolio)
        
        if portfolio_risk < self.cvar_limit:
            penalty = -10.0  # Risk limiti ihlali
        else:
            penalty = 0.0
        
        # Reward = ROI + Risk Penalty + Latency Penalty
        reward = roi + penalty - 0.01 * action.latency
        
        return reward
```

### Kelly Criterion + CVaR Optimization

```python
def kelly_stake(prob, odds, cvar_limit=0.05):
    # Kelly Criterion
    kelly_fraction = (prob * odds - 1) / (odds - 1)
    
    # CVaR kısıtı ile ayarla
    portfolio_var = compute_portfolio_variance()
    
    if portfolio_var > cvar_limit:
        # Risk çok yüksek, stake'i azalt
        adjusted_fraction = kelly_fraction * 0.5
    else:
        adjusted_fraction = kelly_fraction
    
    return max(0, min(adjusted_fraction, 0.25))  # Max %25 bankroll
```

### Dağıtık Eğitim: Ray.io

```python
import ray
from ray import tune

@ray.remote
class RDQLWorker:
    def __init__(self, model_config):
        self.agent = RDQLAgent(model_config)
        
    def train_episode(self, match_data):
        # Sanal simülasyon
        env = PettingZooEnv(match_data)
        
        state = env.reset()
        done = False
        
        while not done:
            # Model inference
            action = self.agent.select_action(state)
            
            # Environment step
            next_state, reward, done, info = env.step(action)
            
            # Experience replay
            self.agent.replay_buffer.add(
                state, action, reward, next_state, done
            )
            
            # Train
            if len(self.agent.replay_buffer) > 1000:
                self.agent.train_batch()
            
            state = next_state
        
        return self.agent.get_metrics()

# Dağıtık eğitim
ray.init()
workers = [RDQLWorker.remote(config) for _ in range(8)]
results = ray.get([w.train_episode.remote(data) for w in workers])
```

## 📊 4. Feast Feature Engineering

### GNN Verisi Saklama (Protobuf)

```python
# GNN graph'ını Protobuf olarak sakla
import protobuf

def encode_graph(nodes, edges):
    graph_proto = GraphProto()
    graph_proto.nodes.extend(nodes.tolist())
    graph_proto.edges.extend(edges.tolist())
    return graph_proto.SerializeToString()

# Feast Feature View
from feast import Feature, FeatureView, Entity

match_entity = Entity(name="match_id", join_key="match_id")

graph_feature_view = FeatureView(
    name="graph_embeddings",
    entities=[match_entity],
    features=[
        Feature(name="graph_blob", dtype="bytes"),  # Protobuf
        Feature(name="dominance", dtype="float"),
        Feature(name="odds", dtype="float")
    ],
    batch_source="clickhouse_source",
    online_store="redis"
)
```

### Historical Features (Eğitim Seti)

```python
class SpatialTemporalDataset(Dataset):
    def __init__(self, feast_client, entity_df):
        # Feast'ten historical features çek
        self.features = feast_client.get_historical_features(
            entity_df=entity_df,
            features=[
                "graph_embeddings:graph_blob",
                "graph_embeddings:dominance",
                "graph_embeddings:odds"
            ]
        ).to_df()
        
    def __getitem__(self, idx):
        # Protobuf decode
        graph_blob = self.features.iloc[idx]['graph_blob']
        graph_tensor = decode_protobuf(graph_blob)
        
        # Global Mean Pooling
        pooled = torch.mean(graph_tensor, dim=0)
        
        # Market features
        market = torch.tensor([
            self.features.iloc[idx]['dominance'],
            self.features.iloc[idx]['odds']
        ])
        
        # Concatenate
        x_t = torch.cat([pooled, market])
        
        return x_t, self.features.iloc[idx]['outcome']
```

## 🎮 5. Sanal Simülasyon Ortamı

### PettingZoo Multi-Agent Environment

```python
from pettingzoo import AECEnv

class BettingEnv(AECEnv):
    def __init__(self, match_data):
        self.match_data = match_data
        self.agents = ["agent_0"]  # Tek agent (sanal betting)
        self.bankroll = 1000.0
        
    def step(self, action):
        # Action: (bet_type, stake, odds)
        bet_type, stake, odds = action
        
        # Simülasyon: Maç sonucunu tahmin et
        outcome = self.simulate_match()
        
        # Reward hesapla
        if outcome == bet_type:
            payout = stake * odds
            reward = payout - stake
        else:
            reward = -stake
        
        # Bankroll güncelle
        self.bankroll += reward
        
        # State güncelle
        next_state = self.get_state()
        
        done = self.bankroll <= 0 or self.match_ended
        
        return next_state, reward, done, {}
```

## 🎯 6. Kritik Teknik Kararlar

### 1. Veri Depolama
- ❌ TimescaleDB (WAL şişmesi, düşük ingestion)
- ✅ ClickHouse (1M/s ingestion, vectorized queries)

### 2. CDC Stratejisi
- ❌ Debezium (ClickHouse native CDC yok)
- ✅ Kafka Engine + Materialized View

### 3. Hudi Upsert
- ❌ Copy-on-Write (write amplification)
- ✅ Merge-on-Read (%60 yazma yükü azaltma)

### 4. Model Seçimi
- ❌ Kalman Filter (lineer varsayım)
- ✅ LSTM-State-Space (non-lineer dynamics)

### 5. Pooling Stratejisi
- ✅ Global Attention Pooling (Mean + Attention hibrit)

### 6. Feature Store
- ✅ Feast (Redis online + Hudi MOR offline)
- ✅ GNN verisi Protobuf/bytes formatında

## 📐 7. Eksiksiz Sistem Mimarisi

```
📦 RDQL SANAL BETTING SYSTEM
│
├── 🌐 DATA INGESTION
│   ├── API Gateway (Pre-match + Live)
│   └── Kafka Topics (prematch, live, odds)
│
├── 💾 STORAGE (3-Tier)
│   ├── ClickHouse (Ana Depo)
│   │   ├── ReplacingMergeTree (odds upsert)
│   │   ├── Materialized Views (rollup)
│   │   └── Kafka Engine (CDC)
│   ├── Feast Feature Store
│   │   ├── Redis (Online - sub-ms)
│   │   └── Delta Lake (Offline - Hudi MOR)
│   └── S3 (Cold Storage - TTL)
│
├── 🔄 STREAM PROCESSING
│   └── Flink CDC
│       ├── ClickHouse Kafka Engine → Flink
│       ├── Redis (Lua CAS version check)
│       └── Delta Lake (Hudi MOR upsert)
│
├── 🧠 AI/ML LAYER
│   ├── Graph-LSTM Encoder
│   │   ├── GNN (Spatial)
│   │   ├── LSTM (Temporal)
│   │   └── Global Attention Pooling
│   ├── LSTM-State-Space Core
│   │   └── Non-linear Dynamics
│   └── TFT Decoder
│       └── Variable Selection + Interpretability
│
├── 🤖 RDQL AGENT
│   ├── CVaR-Kısıtlı Reward
│   ├── Kelly Criterion Stake
│   └── Experience Replay
│
├── 🎮 SIMULATION
│   ├── PettingZoo Environment
│   └── Ray.io (Dağıtık Eğitim)
│
└── 📊 MONITORING
    ├── Prometheus Metrics
    └── Grafana Dashboard
```

## 🚀 Sonuç

Bu sistem:
1. ✅ **Sürekli öğrenir** (RDQL + Experience Replay)
2. ✅ **Gelişir** (Ray.io dağıtık eğitim)
3. ✅ **Strateji geliştirir** (Kelly + CVaR)
4. ✅ **Sanal kazanç elde eder** (PettingZoo simülasyon)
5. ✅ **Canlı veriyi yönetir** (ClickHouse + Flink CDC)
6. ✅ **Yorumlanabilir** (TFT Variable Selection)

**Kritik Başarı Faktörleri:**
- ClickHouse (1M/s ingestion)
- Kafka Engine CDC (native CDC olmadan)
- Hudi MOR (%60 yazma optimizasyonu)
- Graph-LSTM + TFT (spatial-temporal + interpretability)
- LSTM-State-Space (non-lineer dynamics)
- Global Attention Pooling (node → graph)
- CVaR-kısıtlı RDQL (risk yönetimi)

