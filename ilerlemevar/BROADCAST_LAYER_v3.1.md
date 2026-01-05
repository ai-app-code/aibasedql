# 🎙️ SUPERBET GENESIS v3.1 - BROADCAST LAYER
## Süper-Rasyonel Dijital Bahis Varlığı - Yayın Katmanı Mimari Planı

**Oluşturma:** 04.01.2026  
**Kaynak:** TETRA AI Münazara Paneli (4 Tur, 10 LLM, Oy Birliği)  
**Versiyon:** v3.1 (bettingenesis-v3.1.md ile entegre)  
**Durum:** ✅ ONAYLANDI

---

# 📌 BÖLÜM 0: YÖNETİCİ ÖZETİ

## Amaç

SUPERBET GENESIS v3.1 sisteminin ürettiği tahminleri dış dünyaya yayınlamak için **BroadcastPlant** katmanı tasarlandı. Bu katman:

- ✅ Pre-match kuponları yayınlar
- ✅ Canlı pozisyon değişikliklerini yayınlar
- ✅ Platform-agnostic standart format üretir
- ✅ Mevcut sistemi bozmadan entegre olur

## Hedef Platformlar

| Platform | Öncelik | API |
|----------|---------|-----|
| **X (Twitter)** | P0 | Developer API v2 |
| **Telegram** | P1 | Bot API |
| **Android Push** | P2 | Firebase FCM |

## Kritik Metrikler

| Metrik | Hedef |
|--------|-------|
| Yayın Latency | p99 < 60ms |
| Günlük Limit | 200 yayın |
| Filtre Eşiği | Conf>0.65, VSNR>2.2, CAS>1.0 |
| CB Recovery | 15dk Digest Buffer |

---

# 📊 BÖLÜM 1: MİMARİ KONUM

## Sistem Akış Diyagramı

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SUPERBET GENESIS v3.1                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  API-Football v3 → Kafka → Flink → ClickHouse/Redis                        │
│                                          ↓                                  │
│                                   Feast Feature Store                       │
│                                          ↓                                  │
│                                   KServe Inference                          │
│                                          ↓                                  │
│                                   HRL Agents                                │
│                                   (Pre-match DQN / Live LSTM+PPO)           │
│                                          ↓                                  │
│                                   Risk Management                           │
│                                          ↓                                  │
│                              ┌───────────────────────┐                      │
│                              │   🎙️ BROADCAST LAYER  │ ← YENİ EKLENEN      │
│                              │      (BroadcastPlant) │                      │
│                              └───────────┬───────────┘                      │
│                                          ↓                                  │
│                                   Observability                             │
│                                   (Prometheus/Grafana)                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Konum Gerekçesi

| Karar | Gerekçe | Oy |
|-------|---------|-----|
| Risk Management **SONRASI** | Sadece onaylanmış tahminler yayınlanır | 8/8 ✅ |
| Async Sidecar Pattern | Ana akış etkilenmez | 8/8 ✅ |
| Observability ÖNCESİ | Yayın metrikleri Prometheus'a gider | 8/8 ✅ |

---

# 🔄 BÖLÜM 2: BROADCAST LAYER MİMARİSİ

## Detaylı Akış

```
Risk Management
       ↓
       │ Kafka: risk.verified
       ↓
┌───────────────────────────────────────────────────────────────┐
│                      BROADCAST PLANT                          │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐                                      │
│  │  CloudEvents        │  • Filtre: Conf>0.65, VSNR>2.2       │
│  │  Formatter          │  • Transform: Raw + Formatted        │
│  │                     │  • Priority Score hesapla            │
│  └──────────┬──────────┘                                      │
│             ↓                                                 │
│  ┌─────────────────────┐                                      │
│  │  Priority           │  • max_poll_records=5                │
│  │  Dispatcher         │  • manual commit                     │
│  │                     │  • priority header sorting           │
│  └──────────┬──────────┘                                      │
│             ↓                                                 │
│  ┌─────────────────────────────────────────────┐              │
│  │  Kafka Per-Platform Outbox Topics:          │              │
│  │  • broadcast.outbox.twitter                 │              │
│  │  • broadcast.outbox.telegram                │              │
│  │  • broadcast.outbox.android                 │              │
│  └──────────┬──────────────────────────────────┘              │
│             ↓                                                 │
│  ┌─────────────────────────────────────────────┐              │
│  │  Platform Adapters (Consumer Groups):       │              │
│  │  ┌─────────────────────────────────────┐    │              │
│  │  │ TwitterAdapter                      │    │              │
│  │  │ + Circuit Breaker (Resilience4j)    │    │              │
│  │  │ + Full Jitter Retry                 │    │              │
│  │  │ + Token Bucket Rate Limiter         │    │              │
│  │  └─────────────────────────────────────┘    │              │
│  │  ┌─────────────────────────────────────┐    │              │
│  │  │ TelegramAdapter                     │    │              │
│  │  └─────────────────────────────────────┘    │              │
│  │  ┌─────────────────────────────────────┐    │              │
│  │  │ AndroidPushAdapter                  │    │              │
│  │  └─────────────────────────────────────┘    │              │
│  └──────────┬──────────────────────────────────┘              │
│             ↓                                                 │
│        ┌────┴────┐                                            │
│        ↓         ↓                                            │
│   External   Digest Buffer                                    │
│   Platforms  (CB OPEN durumunda)                              │
│                  ↓                                            │
│              15dk Timeout                                     │
│                  ↓                                            │
│              Batch Retry                                      │
│                  ↓                                            │
│              DLQ (Final)                                      │
│                                                               │
└───────────────────────────────────────────────────────────────┘
       ↓
 Prometheus Metrics
```

---

# 📋 BÖLÜM 3: EVENT FORMATI

## CloudEvents v1.0 Standardı

> **Karar:** CloudEvents v1.0 standardı kabul edildi (7/8 oy)  
> **Gerekçe:** Mevcut Kafka pipeline'da `football.match.update` zaten CloudEvents kullanıyor. Interoperability sağlar.

### Tam Payload Şeması

```json
{
  "specversion": "1.0",
  "type": "com.superbet.prediction.published",
  "source": "/plant/broadcast",
  "id": "pred-${match_id}-${timestamp}",
  "datacontenttype": "application/json",
  "time": "2026-01-04T15:30:00Z",
  "data": {
    "match_id": "ENG1_20260104_ARS_LIV",
    "kickoff": "2026-01-04T17:30:00Z",
    "teams": {
      "home": "Arsenal",
      "away": "Liverpool"
    },
    "prediction": {
      "pick": "H",
      "home_win": 0.42,
      "draw": 0.28,
      "away_win": 0.30
    },
    "odds": {
      "home_win": 2.10,
      "draw": 3.40,
      "away_win": 3.50
    },
    "metrics": {
      "confidence": 0.72,
      "vsnr": 2.45,
      "cas": 1.15,
      "kelly_fraction": 0.08,
      "gamma": 0.58,
      "risk_score": 0.03
    },
    "formatted": {
      "twitter": "⚽️ Arsenal vs Liverpool: Ev 42% @2.10 | Conf:72% VSNR:2.45 #SuperbetGenesis",
      "telegram": "🎯 *Arsenal* vs *Liverpool*\n\n🏠 Ev Sahibi: 42% @2.10\n📊 Güven: 72%\n📈 VSNR: 2.45\n🎲 CAS: 1.15\n\n#SuperbetGenesis",
      "android": {
        "title": "Arsenal vs Liverpool",
        "body": "Ev Sahibi Kazanır 42% | Güven: 72%"
      }
    },
    "metadata": {
      "agent_type": "prematch",
      "model_version": "v3.1",
      "priority_score": 0.0864
    }
  }
}
```

### Raw vs Formatted Ayrımı

| Alan | Kullanım | Tüketici |
|------|----------|----------|
| `data.prediction` | Raw probability | Developer API, Analytics |
| `data.metrics` | Raw metrikler | Monitoring, Debugging |
| `data.formatted.twitter` | 280 karakter tweet | TwitterAdapter |
| `data.formatted.telegram` | Markdown mesaj | TelegramAdapter |
| `data.formatted.android` | Push notification | AndroidAdapter |

---

# 🎯 BÖLÜM 4: FİLTRELEME VE ÖNCELİKLENDİRME

## Filtre Eşikleri

> **Karar:** Birleşik filtre eşiği kabul edildi (7/8 oy)

| Metrik | Eşik | Gerekçe |
|--------|------|---------|
| **Confidence** | > 0.65 | Referans 0.6'dan +0.05 gürültü azaltma |
| **VSNR** | > 2.2 | Sinyal gücü garantisi |
| **CAS** | > 1.0 | Adaptasyon skoru minimum |

### Filtre Implementasyonu

```python
class BroadcastFilter:
    """
    TETRA Panel Kararı: Conf>0.65 + VSNR>2.2 + CAS>1.0
    """
    THRESHOLDS = {
        "confidence": 0.65,
        "vsnr": 2.2,
        "cas": 1.0
    }
    
    def should_broadcast(self, prediction: dict) -> tuple[bool, float]:
        metrics = prediction["metrics"]
        
        # Filtre kontrolü
        if metrics["confidence"] < self.THRESHOLDS["confidence"]:
            return False, 0.0
        if metrics["vsnr"] < self.THRESHOLDS["vsnr"]:
            return False, 0.0
        if metrics["cas"] < self.THRESHOLDS["cas"]:
            return False, 0.0
        
        # Priority score hesapla
        priority = self.calculate_priority(metrics)
        return True, priority
```

## Priority Score Formülü

> **Karar:** Logaritmik + Clipping yaklaşımı kabul edildi  
> **Katkılar:** [Alfa] log1p önerisi + [Gamma] clipping stratejisi

### Nihai Formül

```python
import numpy as np

class PriorityScorer:
    """
    Priority Score Formula (TETRA Panel Onaylı):
    
    S = (Conf × Kelly) × clip(log1p(VSNR), 1.0, 1.5) × clip(sqrt(CAS), 1.0, 1.2)
    
    Gerekçeler:
    - log1p(VSNR): Outlier'ların kuyruğu bloke etmesini engeller
    - sqrt(CAS): Doğrusal olmayan, eşik sonrası sınırlı artış
    - Clipping: Uç değerlerin sistem dengesini bozmasını önler
    """
    
    def calculate(self, prediction: dict) -> float:
        metrics = prediction["metrics"]
        
        conf = metrics["confidence"]
        kelly = metrics["kelly_fraction"]
        vsnr = metrics["vsnr"]
        cas = metrics["cas"]
        
        # Base score
        base_score = conf * kelly
        
        # VSNR multiplier (logarithmic + clipped)
        vsnr_mult = min(max(np.log1p(vsnr), 1.0), 1.5)
        
        # CAS multiplier (sqrt + clipped)
        cas_mult = min(max(np.sqrt(cas), 1.0), 1.2)
        
        # Final priority score
        priority_score = base_score * vsnr_mult * cas_mult
        
        return priority_score
```

### Priority Örnekleri

| Conf | Kelly | VSNR | CAS | Priority Score |
|------|-------|------|-----|----------------|
| 0.72 | 0.08 | 2.45 | 1.15 | ~0.086 |
| 0.85 | 0.12 | 3.10 | 1.35 | ~0.141 |
| 0.68 | 0.05 | 2.25 | 1.02 | ~0.042 |

---

# 🔧 BÖLÜM 5: ADAPTER PATTERN

## BaseAdapter Interface

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional

@dataclass
class Result:
    success: bool = False
    retry: bool = False
    reason: Optional[str] = None
    error: Optional[Exception] = None

class BaseAdapter(ABC):
    """
    Tüm platform adapter'ları bu interface'i implemente eder.
    SPI (Service Provider Interface) pattern ile dinamik yükleme.
    """
    
    @abstractmethod
    async def publish(self, cloud_event: dict) -> Result:
        """Publish CloudEvent to target platform"""
        pass
    
    @abstractmethod
    def get_platform_name(self) -> str:
        """Return platform identifier"""
        pass
```

## TwitterAdapter Implementasyonu

```python
import asyncio
import random
from resilience4j import CircuitBreaker

class TwitterAdapter(BaseAdapter):
    """
    X (Twitter) Platform Adapter
    
    Features:
    - Token Bucket Rate Limiting
    - Circuit Breaker (Resilience4j)
    - Full Jitter Exponential Backoff
    """
    
    def __init__(
        self, 
        client: TwitterClient, 
        rate_limiter: PlatformRateLimiter,
        circuit_breaker: CircuitBreaker
    ):
        self.client = client
        self.rate_limiter = rate_limiter
        self.cb = circuit_breaker
        self.backoff = FullJitterBackoff(base=1.0, cap=60.0, max_attempts=5)
    
    def get_platform_name(self) -> str:
        return "twitter"
    
    async def publish(self, event: dict) -> Result:
        # Circuit Breaker check
        if self.cb.state == "OPEN":
            return Result(retry=False, reason="cb_open")
        
        # Rate limiter check
        if not await self.rate_limiter.try_acquire("twitter", 1):
            return Result(retry=True, reason="rate_limited")
        
        try:
            text = event["data"]["formatted"]["twitter"]
            await self.client.tweet(text)
            self.cb.record_success()
            return Result(success=True)
        except Exception as e:
            self.cb.record_failure()
            return Result(success=False, error=e)
    
    async def publish_with_retry(self, event: dict) -> Result:
        """Full Jitter Exponential Backoff ile retry"""
        for attempt in range(self.backoff.max_attempts):
            result = await self.publish(event)
            
            if result.success:
                return result
            
            if not result.retry:
                # CB open veya permanent failure
                return result
            
            # Exponential backoff with full jitter
            wait_time = await self.backoff.calculate_wait(attempt)
            await asyncio.sleep(wait_time)
        
        return Result(retry=False, reason="max_attempts_exceeded")
```

## TelegramAdapter Implementasyonu

```python
class TelegramAdapter(BaseAdapter):
    """Telegram Bot API Adapter"""
    
    def __init__(self, bot: TelegramBot, chat_id: str, rate_limiter: PlatformRateLimiter):
        self.bot = bot
        self.chat_id = chat_id
        self.rate_limiter = rate_limiter
    
    def get_platform_name(self) -> str:
        return "telegram"
    
    async def publish(self, event: dict) -> Result:
        if not await self.rate_limiter.try_acquire("telegram", 1):
            return Result(retry=True, reason="rate_limited")
        
        try:
            text = event["data"]["formatted"]["telegram"]
            await self.bot.send_message(
                chat_id=self.chat_id,
                text=text,
                parse_mode="Markdown"
            )
            return Result(success=True)
        except Exception as e:
            return Result(success=False, error=e)
```

## SPI Registration

```
META-INF/services/
└── com.superbet.broadcast.adapters
    ├── com.superbet.broadcast.adapters.TwitterAdapter
    ├── com.superbet.broadcast.adapters.TelegramAdapter
    └── com.superbet.broadcast.adapters.AndroidPushAdapter
```

---

# ⏱️ BÖLÜM 6: RATE LIMITING

## Token Bucket Stratejisi

> **Karar:** Redis Token Bucket (per-platform) kabul edildi (8/8 oy)

### Platform Limitleri

| Platform | Günlük | Saatlik | Window |
|----------|--------|---------|--------|
| Twitter | 50 | 10 | 1h |
| Telegram | 200 | 30 | 1h |
| Android | 1000 | 100 | 1h |
| **Global** | **200** | - | 24h |

### Implementasyon

```python
import time
import redis

class PlatformRateLimiter:
    """
    Redis Token Bucket Rate Limiter
    Key format: broadcast:limits:{platform}
    """
    
    LIMITS = {
        "twitter": {"daily": 50, "hourly": 10, "refill_rate": 10/3600},
        "telegram": {"daily": 200, "hourly": 30, "refill_rate": 30/3600},
        "android": {"daily": 1000, "hourly": 100, "refill_rate": 100/3600},
        "global": {"daily": 200, "hourly": None, "refill_rate": 200/86400}
    }
    
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
    
    async def try_acquire(self, platform: str, tokens: int = 1) -> bool:
        key = f"broadcast:limits:{platform}"
        limits = self.LIMITS[platform]
        
        # Lua script for atomic token bucket operation
        script = """
        local key = KEYS[1]
        local tokens_requested = tonumber(ARGV[1])
        local max_tokens = tonumber(ARGV[2])
        local refill_rate = tonumber(ARGV[3])
        local now = tonumber(ARGV[4])
        
        local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
        local current_tokens = tonumber(bucket[1]) or max_tokens
        local last_refill = tonumber(bucket[2]) or now
        
        -- Refill tokens
        local elapsed = now - last_refill
        current_tokens = math.min(max_tokens, current_tokens + elapsed * refill_rate)
        
        if current_tokens >= tokens_requested then
            current_tokens = current_tokens - tokens_requested
            redis.call('HMSET', key, 'tokens', current_tokens, 'last_refill', now)
            return 1
        else
            redis.call('HMSET', key, 'tokens', current_tokens, 'last_refill', now)
            return 0
        end
        """
        
        result = await self.redis.eval(
            script,
            1,
            key,
            tokens,
            limits["hourly"],
            limits["refill_rate"],
            time.time()
        )
        
        return result == 1
```

---

# 🛡️ BÖLÜM 7: RESILIENCE MEKANİZMALARI

## Circuit Breaker

> **Karar:** Resilience4j per adapter kabul edildi (8/8 oy)

### Konfigürasyon

| Parametre | Değer | Açıklama |
|-----------|-------|----------|
| Failure Threshold | 50% | Hata oranı eşiği |
| Timeout | 5 dakika | CB OPEN süresi |
| Half-Open Calls | 3 | Test çağrısı sayısı |

### Implementasyon

```python
import asyncio
from enum import Enum
from dataclasses import dataclass
from typing import Callable

class CBState(Enum):
    CLOSED = "CLOSED"
    OPEN = "OPEN"
    HALF_OPEN = "HALF_OPEN"

@dataclass
class CircuitBreakerConfig:
    failure_threshold: float = 0.5
    timeout_seconds: int = 300  # 5 dakika
    half_open_calls: int = 3

class CircuitBreaker:
    """
    TETRA Panel Kararı: Per-adapter Circuit Breaker
    %50 hata oranında OPEN → 5dk bekle → HALF_OPEN → test
    """
    
    def __init__(self, config: CircuitBreakerConfig, digest_buffer: "DigestBuffer"):
        self.config = config
        self.digest_buffer = digest_buffer
        self.state = CBState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.total_count = 0
        self._half_open_count = 0
    
    @property
    def failure_rate(self) -> float:
        if self.total_count == 0:
            return 0.0
        return self.failure_count / self.total_count
    
    async def call(self, adapter: BaseAdapter, event: dict) -> Result:
        if self.state == CBState.OPEN:
            # Digest Buffer'a yönlendir
            await self.digest_buffer.buffer_event(event, adapter.get_platform_name())
            return Result(retry=False, reason="cb_open")
        
        if self.state == CBState.HALF_OPEN:
            if self._half_open_count >= self.config.half_open_calls:
                return Result(retry=False, reason="half_open_limit")
            self._half_open_count += 1
        
        try:
            result = await adapter.publish(event)
            self._record_result(result.success)
            return result
        except Exception as e:
            self._record_result(False)
            raise
    
    def _record_result(self, success: bool):
        self.total_count += 1
        if success:
            self.success_count += 1
            if self.state == CBState.HALF_OPEN:
                self._reset()
        else:
            self.failure_count += 1
            self._check_threshold()
    
    def _check_threshold(self):
        if self.failure_rate > self.config.failure_threshold:
            self.state = CBState.OPEN
            asyncio.create_task(self._schedule_half_open())
    
    async def _schedule_half_open(self):
        await asyncio.sleep(self.config.timeout_seconds)
        self.state = CBState.HALF_OPEN
        self._half_open_count = 0
    
    def _reset(self):
        self.state = CBState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.total_count = 0
```

## Digest Buffer

> **Karar:** DLQ yerine Digest Buffer tercih edildi (7/8 oy)  
> **Gerekçe:** Kullanıcıya bayat spam (DLQ replay) yerine özet gönderimi daha sağlıklı

### Mekanizma

```
CB OPEN → Event Digest Buffer'a düşer
         ↓
     15dk bekle (Redis SETEX TTL=900)
         ↓
     CB CLOSED → Batch retry
         ↓
     Başarısız → DLQ (final)
```

### Implementasyon

```python
import json
import time

class DigestBuffer:
    """
    TETRA Panel Kararı: 15dk Digest Buffer
    
    Key format: broadcast:digest:{platform}:{bucket_id}
    bucket_id = int(timestamp // 900)  # 15 dakikalık bucket'lar
    """
    
    TTL_SECONDS = 900  # 15 dakika
    
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
    
    def _get_bucket_key(self, platform: str) -> str:
        bucket_id = int(time.time() // self.TTL_SECONDS)
        return f"broadcast:digest:{platform}:{bucket_id}"
    
    async def buffer_event(self, event: dict, platform: str):
        """CB OPEN durumunda event'i buffer'a ekle"""
        key = self._get_bucket_key(platform)
        event_json = json.dumps(event)
        
        # RPUSH + EXPIRE (atomic değil ama yeterli)
        await self.redis.rpush(key, event_json)
        await self.redis.expire(key, self.TTL_SECONDS * 2)  # 30dk TTL
    
    async def flush_bucket(self, platform: str, bucket_key: str, adapter: BaseAdapter) -> dict:
        """Bucket'taki tüm event'leri batch olarak gönder"""
        events = await self.redis.lrange(bucket_key, 0, -1)
        
        results = {"success": 0, "failed": 0}
        
        for event_json in events:
            event = json.loads(event_json)
            result = await adapter.publish_with_retry(event)
            
            if result.success:
                results["success"] += 1
            else:
                results["failed"] += 1
                # DLQ'ya gönder
                await self._send_to_dlq(event, platform, result.reason)
        
        # Başarılı flush sonrası bucket'ı sil
        await self.redis.delete(bucket_key)
        
        return results
    
    async def _send_to_dlq(self, event: dict, platform: str, reason: str):
        """Final failure → DLQ"""
        dlq_event = {
            "original_event": event,
            "platform": platform,
            "failure_reason": reason,
            "timestamp": time.time()
        }
        await self.kafka_producer.send("broadcast.dlq", dlq_event)
```

## Full Jitter Exponential Backoff

> **Karar:** Full Jitter stratejisi kabul edildi  
> **Gerekçe:** Thundering herd problemini önler

### Formül

```
wait = min(cap, base × 2^attempt) / 2 + random(0, wait/2)
```

### Implementasyon

```python
import random
import asyncio

class FullJitterBackoff:
    """
    TETRA Panel Kararı: Full Jitter Exponential Backoff
    
    Formula: wait = min(cap, base * 2**attempt) / 2 + random(0, wait/2)
    
    Parameters:
    - base: 1 saniye
    - cap: 60 saniye
    - max_attempts: 5
    """
    
    def __init__(self, base: float = 1.0, cap: float = 60.0, max_attempts: int = 5):
        self.base = base
        self.cap = cap
        self.max_attempts = max_attempts
    
    async def calculate_wait(self, attempt: int) -> float:
        """
        Full Jitter: Thundering herd'ü önler
        
        Attempt 0: 0.5-1.0s
        Attempt 1: 1.0-2.0s
        Attempt 2: 2.0-4.0s
        Attempt 3: 4.0-8.0s
        Attempt 4: 8.0-16.0s (capped at 60s)
        """
        inner = min(self.cap, self.base * (2 ** attempt))
        wait = inner / 2 + random.uniform(0, inner / 2)
        return wait
```

---

# 📡 BÖLÜM 8: KAFKA TOPOLOGY

## Topic Yapısı

> **Karar:** Per-platform outbox topics kabul edildi (7/8 oy)  
> **Gerekçe:** Bulkhead Pattern - X API yavaşlarsa Telegram etkilenmez

```
Kafka Topics:
│
├── risk.verified                   ← Input (Risk Management'dan)
│   └── Producer: RiskManagementPlant
│   └── Consumer: broadcast-plant-formatter
│
├── broadcast.queue.priority        ← Internal (Priority Dispatcher)
│   └── Headers: priority_score, platform
│   └── Consumer: broadcast-plant-dispatcher
│
├── broadcast.outbox.twitter        ← Output (Platform-specific)
│   └── Consumer Group: twitter-adapter-consumer
│
├── broadcast.outbox.telegram       ← Output
│   └── Consumer Group: telegram-adapter-consumer
│
├── broadcast.outbox.android        ← Output
│   └── Consumer Group: android-adapter-consumer
│
├── broadcast.digest.buffer         ← CB Fallback (Internal)
│   └── Consumer: digest-flusher (cron: every 15min)
│
└── broadcast.dlq                   ← Dead Letter Queue (Final)
    └── Consumer: dlq-processor (manual review)
```

## Consumer Konfigürasyonu

```python
KAFKA_CONSUMER_CONFIG = {
    "bootstrap.servers": "kafka:9092",
    "group.id": "broadcast-plant-priority",
    "auto.offset.reset": "earliest",
    "enable.auto.commit": False,     # TETRA Kararı: Manual commit
    "max.poll.records": 5,           # TETRA Kararı: Starvation önleme
    "fetch.min.bytes": 1024,
    "fetch.max.wait.ms": 500,
    "receive.buffer.bytes": 65536
}
```

## Priority Consumer

```python
class PriorityConsumer:
    """
    Priority header'a göre sıralama + batch processing
    max_poll_records=5 ile starvation önlenir
    """
    
    async def consume(self):
        messages = await self.consumer.poll(timeout_ms=1000)
        
        # Priority'ye göre sırala
        prioritized = []
        for topic_partition, records in messages.items():
            for record in records:
                priority = float(record.headers.get("priority", 0.0))
                prioritized.append((priority, record))
        
        # Yüksek priority önce
        prioritized.sort(key=lambda x: -x[0])
        
        # Batch process
        batch = prioritized[:5]
        
        results = await asyncio.gather(*[
            self.process_message(record) 
            for _, record in batch
        ])
        
        # Manual commit
        await self.consumer.commit()
```

---

# 📊 BÖLÜM 9: MONITORING VE ALERTING

## Prometheus Metrics

```yaml
# Counter metrics
broadcast_published_total{platform="twitter", priority="high"}
broadcast_published_total{platform="telegram", priority="low"}
broadcast_dropped_total{reason="filter_low_confidence"}
broadcast_dropped_total{reason="filter_low_vsnr"}
broadcast_dropped_total{reason="filter_low_cas"}
broadcast_dropped_total{reason="rate_limited"}

# Gauge metrics
broadcast_circuit_breaker_state{platform="twitter"}  # 0=CLOSED, 1=OPEN, 2=HALF_OPEN
broadcast_rate_limit_tokens{platform="twitter"}
broadcast_digest_buffer_size{platform="twitter"}
broadcast_digest_staleness_seconds{platform="twitter"}

# Histogram metrics
broadcast_latency_seconds{platform="twitter", quantile="0.5"}
broadcast_latency_seconds{platform="twitter", quantile="0.99"}

# Error metrics
broadcast_error_rate{platform="twitter"}
broadcast_retry_count{platform="twitter"}
```

## Alert Rules

```yaml
groups:
  - name: broadcast_alerts
    rules:
      # Digest Buffer Staleness Alert
      - alert: DigestBufferStale
        expr: broadcast_digest_staleness_seconds > 900
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Digest Buffer {{ $labels.platform }} stale"
          description: "Digest Buffer for {{ $labels.platform }} has messages older than 15 minutes"
      
      # Circuit Breaker Open Alert
      - alert: CircuitBreakerOpen
        expr: broadcast_circuit_breaker_state == 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Circuit Breaker OPEN for {{ $labels.platform }}"
      
      # High Error Rate Alert
      - alert: BroadcastHighErrorRate
        expr: broadcast_error_rate > 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High broadcast error rate for {{ $labels.platform }}"
      
      # Rate Limit Approaching
      - alert: RateLimitApproaching
        expr: broadcast_rate_limit_tokens < 5
        for: 1m
        labels:
          severity: info
        annotations:
          summary: "Rate limit tokens low for {{ $labels.platform }}"
```

## Grafana Dashboard Panelleri

| Panel | Metrik | Görselleştirme |
|-------|--------|----------------|
| Yayın/Saat | `rate(broadcast_published_total[1h])` | Time series |
| Platform Dağılımı | `broadcast_published_total by platform` | Pie chart |
| CB Durumu | `broadcast_circuit_breaker_state` | Stat |
| Drop Nedenleri | `broadcast_dropped_total by reason` | Bar chart |
| Latency P99 | `broadcast_latency_seconds{quantile="0.99"}` | Gauge |
| Digest Buffer | `broadcast_digest_buffer_size` | Time series |

---

# 🔗 BÖLÜM 10: bettingenesis-v3.1 ENTEGRASYONU

## Mevcut Sistem Bileşenleri ile İlişki

| v3.1 Bileşeni | Broadcast Entegrasyonu | Notlar |
|---------------|------------------------|--------|
| **HRL Agents** | Tahmin üretir → `risk.verified` topic | Pre-match DQN + Live LSTM+PPO |
| **Risk Management** | Onay sonrası broadcast'e gönderir | VaR, CVaR, Kelly kontrolleri |
| **VSNR** | Filtre eşiği: > 2.2 | Priority score çarpanı |
| **CAS** | Filtre eşiği: > 1.0 | Priority score çarpanı |
| **Confidence** | Filtre eşiği: > 0.65 | Ana filtre kriteri |
| **Kelly Fraction** | Priority score | `Conf × Kelly` base score |
| **Gamma (γ)** | CloudEvents data'da expose | Liderlik/Eşgüdüm modu bilgisi |
| **Circuit Breaker** | Per-adapter Resilience4j | Mevcut CB pattern ile uyumlu |
| **Prometheus** | Broadcast metrikleri eklendi | Mevcut monitoring altyapısı |

## Yeni Kafka Topics

Mevcut topic'lere ek olarak:

```
# Mevcut (v3.1)
prematch, live, odds, graph_events, sentiment

# Yeni (Broadcast Layer)
risk.verified            ← Risk Management çıkışı
broadcast.queue.priority ← Internal priority queue
broadcast.outbox.twitter ← Twitter adapter
broadcast.outbox.telegram← Telegram adapter
broadcast.outbox.android ← Android adapter
broadcast.digest.buffer  ← CB fallback
broadcast.dlq            ← Dead letter queue
```

## Handover Protocol Entegrasyonu

```python
# Pre-match tahminleri yayınlanır
class PreMatchAgent:
    def produce_prediction(self):
        prediction = self.dqn.predict(state)
        # Risk Management'a gönder
        await kafka.send("risk.verified", prediction)
        # → BroadcastPlant → Twitter'da "⚽️ Arsenal vs Liverpool..." tweet'i

# Live tahminleri de yayınlanır
class LiveAgent:
    def on_live_event(self, event):
        updated_prediction = self.lstm_ppo.update(event)
        if self.should_update_position(updated_prediction):
            await kafka.send("risk.verified", updated_prediction)
            # → BroadcastPlant → Twitter'da "🔄 Pozisyon güncellendi..." tweet'i
```

---

# 📋 BÖLÜM 11: MÜNAZARA ÖZETİ

## Katılımcılar ve Katkıları

| Katılımcı | Rol | Ana Katkı |
|-----------|-----|-----------|
| **Nexus** | Baş Mimar/Moderatör | Münazara yönetimi, oylama koordinasyonu |
| **DevOps** | xAI | Tek topic savunusu (azınlık), Kafka config |
| **Alfa** | Google | log1p VSNR önerisi, CloudEvents desteği |
| **Beta** | Google | Bulkhead Pattern, Full Jitter önerisi |
| **Gamma** | OpenAI | Clipping stratejisi, Priority Score birleştirme |
| **Delta** | MoonshotAI | ArgoCD entegrasyonu, Staleness alert |
| **Epsilon** | Qwen | SLO uyumu, mevcut metrik kullanımı |
| **Zeta** | Z.AI | Digest Buffer UX faydaları |
| **Theta** | OpenAI | Oylama sonuçları açıklaması |
| **Kappa** | MiniMax | Final Blueprint hazırlığı |

## Oylama Sonuçları

| Karar | Oy | Sonuç |
|-------|-----|-------|
| Mimari Konum: Risk Mgmt SONRASI | 8/8 | ✅ Kabul |
| Adapter Pattern: BaseAdapter | 8/8 | ✅ Kabul |
| Rate Limiting: Redis Token Bucket | 8/8 | ✅ Kabul |
| Topic: Per-platform outbox | 7/8 | ✅ Kabul |
| Format: CloudEvents v1.0 | 7/8 | ✅ Kabul |
| Filtre: Conf>0.65+VSNR>2.2+CAS>1.0 | 7/8 | ✅ Kabul |
| CB Fallback: Digest Buffer | 7/8 | ✅ Kabul |
| Priority Score: log1p+sqrt+clip | Konsensüs | ✅ Kabul |
| Retry: Full Jitter | Konsensüs | ✅ Kabul |

## Çözülen Çatışmalar

| Çatışma | Pozisyon A | Pozisyon B | Kazanan |
|---------|------------|------------|---------|
| Topic mimarisi | Tek topic (DevOps) | Per-platform (Çoğunluk) | Per-platform |
| CB Fallback | DLQ (DevOps) | Digest Buffer (Çoğunluk) | Digest Buffer |
| Priority Score | Lineer (Beta) | Logaritmik (Alfa) | Logaritmik+Clipping |

---

# 🎯 BÖLÜM 12: SONUÇ VE SONRAKI ADIMLAR

## Tamamlanan Kararlar

- ✅ Broadcast Layer mimarisi tanımlandı
- ✅ CloudEvents v1.0 formatı belirlendi
- ✅ Filtre eşikleri kesinleşti
- ✅ Priority Score formülü onaylandı
- ✅ Kafka topology tasarlandı
- ✅ Resilience mekanizmaları belirlendi
- ✅ Monitoring metrikleri tanımlandı

## Sonraki Adımlar

| Adım | Öncelik | Tahmini Süre |
|------|---------|--------------|
| X Developer hesabı açma | P0 | 1 gün |
| TwitterAdapter POC | P0 | 3 gün |
| BroadcastPlant service skeleton | P0 | 2 gün |
| Kafka topic'leri oluşturma | P1 | 1 gün |
| Redis rate limiter kurulumu | P1 | 1 gün |
| Circuit Breaker implementasyonu | P1 | 2 gün |
| Prometheus metrikleri ekleme | P2 | 1 gün |
| Grafana dashboard | P2 | 1 gün |
| TelegramAdapter | P3 | 2 gün |
| AndroidPushAdapter | P3 | 2 gün |

## İlk MVP Scope

**Hedef:** X (Twitter) platformuna tek tip tahmin yayını

1. BroadcastPlant service
2. TwitterAdapter
3. Temel filtreleme
4. Rate limiting
5. Prometheus metrikleri

---

# 📚 REFERANSLAR

| Dosya | İlişki |
|-------|--------|
| `bettingenesis-v3.1.md` | Ana sistem mimarisi |
| `PROJECT_TREE_v3.1.md` | Proje yapısı |
| `FRONTEND_ARCHITECTURE_v1.0.md` | Frontend blueprint |
| `BROADCAST_MUNAZARA_PROMPT.md` | Münazara promptu |

---

**Broadcast Layer Blueprint v1.0 - ONAYLANDI ✅**

*TETRA AI Münazara Paneli - 04.01.2026*
