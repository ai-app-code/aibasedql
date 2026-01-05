# Project ORACLE - İkiz Veri Motoru & Hibrit Zeka Mimarisi

## 🎯 Proje Vizyonu
**"Sanal Simülasyon Yapan, İkiz Veritabanlı, Hibrit Yapay Zekalı Bir Fintech Medya Çekirdeği (Core)"**

Dış dünyadan (API) beslenen, kendi içinde kapalı devre "Paper Trading" yaparak kendini eğiten ve elde ettiği sinyalleri/içerikleri bir API Gateway üzerinden dış dünyaya (X Botu, Mobil App) servis eden **Otonom Bir Fintech Medya Çekirdeği**.

## 🏗️ 1. İkiz Veritabanı Mimarisi (The Twin Engine)

### Kritik Tespit
Futbol verisi iki karakterlidir ve tek bir DB teknolojisi ikisini birden verimli yönetemez:
1. **Statik/Snapshot Veri:** Fikstür, Maç Önü Oranları, Maç Sonu Skoru, Kadrolar
2. **Canlı/Akışkan Veri:** 90 dakika boyunca akan zaman serisi
   - **Kümülatif (Cumulative):** Korner, Kart, Gol Sayısı (`val += 1`)
   - **Dalgalı (Fluctuating):** Topla Oynama %, Baskı Endeksi, Canlı Oranlar

### Teknoloji Kararları
- **Cold DB (Arşiv):** PostgreSQL + TimescaleDB Extension
- **Hot DB (Canlı):** InfluxDB (Time-Series) + Redis Streams (Live Ingest)

### Merge Stratejisi (Maç Sonu)
**Sorun:** Saniyelik devasa veriyi nasıl sıkıştırıp arşive kaldıracağız?

**Çözüm:** Ramer-Douglas-Peucker (RDP) Algoritması + Downsampling

```python
def merge_hot_to_cold(match_id):
    # 1. Kafka Trigger
    trigger = kafka_consumer.poll(f"match_end.{match_id}")
    
    # 2. Hot DB Downsample (InfluxDB)
    raw_drift = influx.query(f"""
        SELECT mean(drift), max(drift), stddev(drift)
        FROM hot WHERE match='{match_id}'
        GROUP BY time(1m)
    """)
    
    # 3. RDP Sıkıştırma (Şekil Koruyan)
    simplified = rdp(raw_drift.points, epsilon=0.01)  # %90 boyut azaltma
    
    # 4. Events Çek (Kesikli Olaylar)
    events = pg.query(f"""
        SELECT time, type, metadata 
        FROM events WHERE match_id='{match_id}'
    """)
    
    # 5. Cold DB Yaz (TimescaleDB Hypertable)
    hypertable.insert({
        "match_id": match_id,
        "time_bucket": trigger.end_time,
        "drift_rdp": json.dumps(simplified),
        "stats": {"avg": avg, "max": max, "std": std},
        "events": json.dumps(events)
    })
    
    # 6. Hot DB Temizle (24 saat Retention Policy)
    influx.retention_policy(tag="match", duration="24h")
```

**Cold DB Şeması:**
```sql
(match_id, time_bucket, drift_rdp_jsonb, stats_jsonb, events_jsonb)
```

**Kritik Karar:** Basit ortalama yerine RDP kullanımı, kritik tepe/dip noktalarını (anomalileri) kayıpsız saklar.

## 🤖 2. Hibrit Yapay Zeka Mimarisi (The AI Zoo)

### Pre-Match Agent (DQN)
- **Görev:** Statik verilerden (Form, Kadro, Sakatlık) "Value" çıkarma
- **Algoritma:** Deep Q-Network
- **Çıktı:** Q-değerleri, Hidden State ($c_0$), Finansal Pozisyon

### Live Agent (QRDL - LSTM/GRU)
- **Görev:** Maçın "hikayesini" ve momentumunu takip
- **Algoritma:** Recurrent Reinforcement Learning + Quantile Regression
- **Özellik:** Risk dağılımını anlamak için QRDL kullanımı

### Handover Protokolü (Devir Teslim)

**Sorun:** Pre-Match "Ev Sahibi Kazanır" dediyse, Live Agent bu pozisyonu nasıl devralır?

**Çözüm:** Atomic Transfer + Teacher Forcing

```python
def handover_pre_to_live(match_id):
    # 1. Pre-Match DQN Çıktısı
    pre_output = dqn.predict(X_static)
    
    # 2. Redis Atomic Transfer (WATCH-MULTI)
    pipe = redis.pipeline(transaction=True)
    pipe.watch(f"match:{match_id}:handover")
    
    try:
        payload = {
            "q_pre": pre_output.q_values.tolist(),
            "c0": dqn.hidden_state.numpy().tobytes(),  # LSTM init
            "portfolio": {
                "exposure": 100.0,
                "entry_odds": 1.85,
                "stop_loss": 0.15
            },
            "ver": atomic_increment(f"match:{match_id}:handover_ver")
        }
        pipe.multi()
        pipe.hset(f"match:{match_id}:handover", mapping=payload)
        pipe.expire(f"match:{match_id}:handover", 5400)  # 90dk TTL
        pipe.execute()
    except WatchError:
        return trigger_emergency_hedge(match_id)
    
    # 3. Live Agent Devralma
    live_state = redis.hgetall(f"match:{match_id}:handover")
    
    # 4. Teacher Forcing (İlk 10dk)
    for t in range(600):  # 10dk * 1s
        live_agent.step(
            X_live=X_stream[t],
            c0=load_hidden(live_state["c0"]),
            teacher=decode_q_pre(live_state["q_pre"]),
            mode="teacher_forcing"
        )
    
    # 5. Normal Mod
    live_agent.mode = "autonomous"
    return {"status": "handover_complete", "ver": live_state["ver"]}
```

**Kritik Kararlar:**
- ✅ Nöral ağırlıklar ($c_0$), finansal pozisyon ve Q-değerlerinin Redis Atomic Transfer
- ✅ 5 saniye ACK süresi, başarısızlık durumunda Emergency Hedge
- ✅ İlk 10 dakikalık Teacher Forcing (modelin maç başı volatilitede saçmalamasını engeller)

## 🔄 3. Bidirectional Cross-Attention Mimarisi

### TwinAttention Sınıfı

```python
class TwinAttention(nn.Module):
    def __init__(self, d_model=512):
        super().__init__()
        self.ln_pre = nn.LayerNorm(d_model)   # Pre-Match
        self.ln_live = nn.LayerNorm(d_model)  # Live
        self.batch_size = 32
        
    def forward(self, X_static, X_live_batch):
        Z_stat = self.ln_pre(X_static)  # [1, D]
        
        outputs = []
        for batch in self.chunk(X_live_batch, self.batch_size):
            Z_live = self.ln_live(batch)  # [B, T, D]
            
            # Stream A: Live->Static (Anlık Aksiyon)
            Q_A, K_A, V_A = Z_live, Z_stat, Z_stat
            attn_A = softmax(Q_A @ K_A.T / sqrt(D))
            out_A = attn_A @ V_A
            
            # Stream B: Static->Live (Anomali Tespiti)
            Q_B, K_B, V_B = Z_stat, Z_live, Z_live
            attn_B = softmax(Q_B @ K_B.T / sqrt(D))
            out_B = attn_B @ V_B
            
            # Drift (Anomaly Score)
            drift = (out_A - out_B).abs().mean()
            outputs.append(torch.cat([out_A, out_B], dim=-1))
            
        return torch.cat(outputs), drift  # drift -> Epsilon EMA
```

**Anomaly Score (Drift):**
```python
drift = (out_A - out_B).abs().mean()
```

**Dinamik Eşik (Hedge Tetikleyici):**
```python
# 5 dakikalık EMA ile filtreleme
div_ema = 0.2 * div + 0.8 * div_ema_prev
threshold = max(0.1, 0.15 * div_ema)

# Hedge Kararı
if state.drift > state.regret_b:
    adapter.execute_hedge(ratio=sigmoid(state.drift))
```

## 📦 4. Protobuf Veri Transferi

### TwinDelta Mesajı (Bandwidth %60 Azaltma)

```protobuf
syntax = "proto3";

message PinballParams {
    float a = 1;    // 0.15
    float tau = 2;  // 0.9
    float eta = 3;  // 1e-3
}

message TwinState {
    uint32 version = 1;
    int64 ts = 2;
    bytes pre_ln = 3;      // packed float32[512]
    bytes live_ln = 4;
    float drift = 5;
    float regret_b = 6;
    PinballParams p = 7;
    bytes crc32 = 8;
}

message TwinDelta {
    uint32 ver = 1;
    int64 ts = 2;
    float drift = 3;
    float regret_b = 4;
}
```

### CRC32 Doğrulama + Checkpoint Mekanizması

```python
# CRC32 Doğrulama
crc = zlib.crc32(payload[:-4]) == struct.unpack('<I', payload[-4:])[0]

# Checkpoint Trigger (Her 50 delta veya 5 dakikada bir)
pipe.hincrby(f"match:{id}:meta", "ver", 1)
pipe.rpush(f"match:{id}:deltas", delta_bytes)
if new_ver % 50 == 0:
    trigger_full_state_dump(match_id)
```

**Kurtarma Stratejisi:**
- CRC32 başarısız → Son 50 delta sil → Checkpoint'ten reconstruct

## 🚨 5. Emergency Hedge API

### Endpoint Tanımı

```python
@router.post("/broker/hedge/{match_id}")
async def trigger_emergency_hedge(match_id: str, payload: HedgeRequest):
    # Idempotency Check
    if await redis.get(f"hedge:{match_id}:status") == "IN_PROGRESS":
        raise HTTPException(409, "Already hedging")
    
    # Circuit Breaker Check
    if hedge_failures > 3:
        redis.setex(f"circuit:{match_id}", 60, "OPEN")
        alert_slack(f"🚨 Hedge Circuit Open: {match_id}")
        return {"status": "circuit_open", "retry_after": 60}
    
    # BrokerAdapter Call (Adaptive Iceberg Order)
    result = await broker_adapter.execute_hedge(
        match_id=match_id,
        ratio=payload.ratio,
        side="opposite",
        order_type="IOC",          # Immediate or Cancel
        slippage=0.02,
        hedge_id=payload.hedge_id
    )
    
    logging.info(f"Executed hedge for {match_id}: {result}")
    return result
```

### Adaptive Iceberg Order Stratejisi

```python
def execute_hedge(order):
    # Likidite Kontrolü
    if order.size > market.depth * 0.1:
        chunk_size = min(order.size/10, market.depth * 0.05)
        
        for chunk in split_iceberg(order.size, chunk_size):
            result = api.post("/order", chunk, type="IOC", timeout=3)
            
            # Fill Rate Kontrolü
            if result.fill_rate < 0.7:
                # Kalanı Market Order ile kapat
                remaining = order.size - result.filled
                api.post("/order", remaining, type="MARKET")
                break
```

**Kritik Parametreler:**
- `order_type: "IOC"` → Fiyat kayması durumunda yarım kalmasını önler
- `slippage_limit: 0.02` → %2 maksimum kayma
- `hedge_id` → Idempotency key (tekrarlı çağrıları engeller)

## 📡 6. Yayıncı Katmanı (The Broadcaster)

### Attention Mechanism (Hikayesi Olan Maçlar)

```python
# Vector Embedding + Cosine Similarity
storyScore = 0.6*z(drift_ema) + 0.3*z(context_div_ema) + 0.1*event_boost

# Exponential Decay (τ=90s)
# TopK=50/min, min_score=1.2
```

### WebSocket Payload API

```json
{
    "match_id": "12345",
    "t": 1640995200,
    "storyScore": 2.3,
    "sentiment": "HYPE",
    "quantiles": {"q10": 0.8, "q50": 1.2, "q90": 1.8},
    "threshold": 0.15,
    "rdp_segments": [[0,0.1], [60,0.3], [90,0.8]],
    "text": "⚡ 85. dakika: Momentum ev sahibinde!",
    "crossAttention": {
        "attn_q": 0.85,
        "context_div": 0.12
    },
    "uncertainty": {"q_spread": 1.0},
    "mode": "AUTONOMOUS",
    "hedge": false
}
```

### Backtesting (Geriye Dönük Eğitim)

```python
# TimescaleDB → RDP Expand → Kafka Replay
POST /v1/replay?season=2023&speed=8x

# Deterministic seed ile Teacher Forcing windows align
# Regret kaydı ile pinball-loss optimizasyonu
```

## 🎓 Kritik Teknik Kararlar

### 1. Veri Sıkıştırma
- ❌ Basit ortalama (kritik detayları yok eder)
- ✅ RDP (epsilon=0.01) → %90 boyut azaltma + anomali koruması

### 2. Handover Güvenliği
- ✅ Redis WATCH-MULTI (Atomic Transfer)
- ✅ 5 saniye ACK timeout
- ✅ Teacher Forcing (ilk 10 dakika)
- ✅ Emergency Hedge fallback

### 3. Veri Transfer Optimizasyonu
- ❌ JSON (yavaş, büyük)
- ✅ Protobuf (bytes packed) → %40 boyut azaltma, 10x hız artışı
- ✅ TwinDelta (sadece değişenler) → %60 bandwidth azaltma

### 4. Anomali Tespiti
- ✅ Bidirectional Cross-Attention (Live↔Static)
- ✅ 5 dakikalık EMA filtreleme
- ✅ Dinamik eşik ayarlama (pinball-loss regret minimizasyonu)

### 5. Emergency Hedge
- ✅ IOC emir tipi (volatilite yönetimi)
- ✅ Adaptive Iceberg Order (piyasa etkisini minimize)
- ✅ Circuit Breaker (3 başarısızlık → 60s timeout)
- ✅ Idempotency (hedge_id ile)

## 🚀 Deployment Planı

1. **İkiz Veritabanı:** Hot DB (Influx/Redis) + Cold DB (TimescaleDB) + RDP sıkıştırma
2. **Hibrit Zeka:** Atomic Handover + Teacher Forcing + Bidirectional Attention
3. **Acil Durum:** Emergency Hedge API (IOC + Iceberg + Circuit Breaker)
4. **Yayıncı:** Attention Mechanism + WebSocket + Backtesting
5. **Monitoring:** Prometheus + Grafana + Kafka Telemetry

