# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 5
## Broadcast Layer (4+ Seviye Derinlik)

**Kaynak:** BROADCAST_LAYER_v3.1.md (TETRA Münazara Kararları)  
**Oluşturma:** 04.01.2026

---

```
superbet-genesis/ (devam)
│
├── 📁 services/
│   │
│   └── 📁 broadcast-plant/
│       │
│       ├── 📁 src/
│       │   │
│       │   ├── 📁 core/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │
│       │   │   ├── 📄 broadcast_plant.py
│       │   │   │   ├── class BroadcastPlant:
│       │   │   │   │   ├── __init__(kafka_config, redis_client, adapters)
│       │   │   │   │   ├── start() -> None
│       │   │   │   │   │   ├── """BroadcastPlant ana döngüsü"""
│       │   │   │   │   │   ├── consumer = KafkaConsumer("risk.verified")
│       │   │   │   │   │   └── while True: process_message()
│       │   │   │   │   ├── process_message(message: CloudEvent) -> Result
│       │   │   │   │   │   ├── # 1. Filtrele
│       │   │   │   │   │   ├── if not filter.should_broadcast(message):
│       │   │   │   │   │   │   └── return Result(dropped=True)
│       │   │   │   │   │   ├── # 2. Priority hesapla
│       │   │   │   │   │   ├── priority = scorer.calculate(message)
│       │   │   │   │   │   ├── # 3. Format
│       │   │   │   │   │   ├── formatted = formatter.format(message)
│       │   │   │   │   │   ├── # 4. Dispatch
│       │   │   │   │   │   └── return dispatcher.dispatch(formatted, priority)
│       │   │   │   │   ├── shutdown() -> None
│       │   │   │   │   └── health_check() -> HealthStatus
│       │   │   │   └── CONSUMER_GROUP = "broadcast-plant-main"
│       │   │   │
│       │   │   ├── 📄 filter.py
│       │   │   │   ├── class BroadcastFilter:
│       │   │   │   │   ├── """
│       │   │   │   │   ├── TETRA Panel Kararı:
│       │   │   │   │   ├──   Confidence > 0.65
│       │   │   │   │   ├──   VSNR > 2.2
│       │   │   │   │   ├──   CAS > 1.0
│       │   │   │   │   ├── """
│       │   │   │   │   ├── THRESHOLDS = {
│       │   │   │   │   │   ├── "confidence": 0.65,
│       │   │   │   │   │   ├── "vsnr": 2.2,
│       │   │   │   │   │   └── "cas": 1.0
│       │   │   │   │   ├── }
│       │   │   │   │   ├── __init__(thresholds: dict = None)
│       │   │   │   │   ├── should_broadcast(prediction: dict) -> tuple[bool, str]
│       │   │   │   │   │   ├── metrics = prediction["metrics"]
│       │   │   │   │   │   ├── if metrics["confidence"] < THRESHOLDS["confidence"]:
│       │   │   │   │   │   │   └── return False, "low_confidence"
│       │   │   │   │   │   ├── if metrics["vsnr"] < THRESHOLDS["vsnr"]:
│       │   │   │   │   │   │   └── return False, "low_vsnr"
│       │   │   │   │   │   ├── if metrics["cas"] < THRESHOLDS["cas"]:
│       │   │   │   │   │   │   └── return False, "low_cas"
│       │   │   │   │   │   └── return True, "passed"
│       │   │   │   │   └── get_drop_metrics() -> dict
│       │   │   │   └── DROP_REASONS = ["low_confidence", "low_vsnr", "low_cas"]
│       │   │   │
│       │   │   ├── 📄 priority_scorer.py
│       │   │   │   ├── import numpy as np
│       │   │   │   ├── class PriorityScorer:
│       │   │   │   │   ├── """
│       │   │   │   │   ├── TETRA Panel Kararı - Priority Score Formülü:
│       │   │   │   │   ├── S = (Conf × Kelly) × clip(log1p(VSNR), 1.0, 1.5) × clip(sqrt(CAS), 1.0, 1.2)
│       │   │   │   │   ├── 
│       │   │   │   │   ├── Gerekçeler:
│       │   │   │   │   ├──   - log1p(VSNR): Outlier'ların kuyruğu bloke etmesini engeller
│       │   │   │   │   ├──   - sqrt(CAS): Doğrusal olmayan, eşik sonrası sınırlı artış
│       │   │   │   │   ├──   - Clipping: Uç değerlerin sistem dengesini bozmasını önler
│       │   │   │   │   ├── """
│       │   │   │   │   ├── VSNR_CLIP = (1.0, 1.5)
│       │   │   │   │   ├── CAS_CLIP = (1.0, 1.2)
│       │   │   │   │   ├── calculate(prediction: dict) -> float
│       │   │   │   │   │   ├── metrics = prediction["metrics"]
│       │   │   │   │   │   ├── conf = metrics["confidence"]
│       │   │   │   │   │   ├── kelly = metrics["kelly_fraction"]
│       │   │   │   │   │   ├── vsnr = metrics["vsnr"]
│       │   │   │   │   │   ├── cas = metrics["cas"]
│       │   │   │   │   │   ├── # Base score
│       │   │   │   │   │   ├── base_score = conf * kelly
│       │   │   │   │   │   ├── # VSNR multiplier (logarithmic + clipped)
│       │   │   │   │   │   ├── vsnr_mult = min(max(np.log1p(vsnr), 1.0), 1.5)
│       │   │   │   │   │   ├── # CAS multiplier (sqrt + clipped)
│       │   │   │   │   │   ├── cas_mult = min(max(np.sqrt(cas), 1.0), 1.2)
│       │   │   │   │   │   ├── # Final priority score
│       │   │   │   │   │   └── return base_score * vsnr_mult * cas_mult
│       │   │   │   │   └── get_priority_tier(score: float) -> str
│       │   │   │   │       ├── if score > 0.12: return "critical"
│       │   │   │   │       ├── if score > 0.08: return "high"
│       │   │   │   │       ├── if score > 0.05: return "medium"
│       │   │   │   │       └── return "low"
│       │   │   │   └── PRIORITY_TIERS = ["critical", "high", "medium", "low"]
│       │   │   │
│       │   │   ├── 📄 formatter.py
│       │   │   │   ├── from abc import ABC, abstractmethod
│       │   │   │   ├── class CloudEventFormatter:
│       │   │   │   │   ├── """
│       │   │   │   │   ├── CloudEvents v1.0 standardı (TETRA Panel Kararı: 7/8 oy)
│       │   │   │   │   ├── """
│       │   │   │   │   ├── SPEC_VERSION = "1.0"
│       │   │   │   │   ├── TYPE_PREFIX = "com.superbet.prediction"
│       │   │   │   │   ├── SOURCE = "/plant/broadcast"
│       │   │   │   │   ├── __init__(template_engine: TemplateEngine)
│       │   │   │   │   ├── format(prediction: dict, priority: float) -> CloudEvent
│       │   │   │   │   │   ├── return {
│       │   │   │   │   │   │   ├── "specversion": "1.0",
│       │   │   │   │   │   │   ├── "type": f"{TYPE_PREFIX}.published",
│       │   │   │   │   │   │   ├── "source": SOURCE,
│       │   │   │   │   │   │   ├── "id": f"pred-{match_id}-{timestamp}",
│       │   │   │   │   │   │   ├── "datacontenttype": "application/json",
│       │   │   │   │   │   │   ├── "time": datetime.utcnow().isoformat(),
│       │   │   │   │   │   │   └── "data": self._build_data(prediction)
│       │   │   │   │   │   └── }
│       │   │   │   │   ├── _build_data(prediction: dict) -> dict
│       │   │   │   │   │   ├── return {
│       │   │   │   │   │   │   ├── "match_id": prediction["match_id"],
│       │   │   │   │   │   │   ├── "teams": prediction["teams"],
│       │   │   │   │   │   │   ├── "prediction": prediction["prediction"],
│       │   │   │   │   │   │   ├── "odds": prediction["odds"],
│       │   │   │   │   │   │   ├── "metrics": prediction["metrics"],
│       │   │   │   │   │   │   ├── "formatted": self._format_all_platforms(prediction),
│       │   │   │   │   │   │   └── "metadata": {...}
│       │   │   │   │   │   └── }
│       │   │   │   │   └── _format_all_platforms(prediction: dict) -> dict
│       │   │   │   │       ├── return {
│       │   │   │   │       │   ├── "twitter": self._format_twitter(prediction),
│       │   │   │   │       │   ├── "telegram": self._format_telegram(prediction),
│       │   │   │   │       │   └── "android": self._format_android(prediction)
│       │   │   │   │       └── }
│       │   │   │   │
│       │   │   │   ├── class TwitterFormatter:
│       │   │   │   │   ├── MAX_LENGTH = 280
│       │   │   │   │   ├── format(prediction: dict) -> str
│       │   │   │   │   │   ├── template = "⚽️ {home} vs {away}: {pick} {prob}% @{odds} | Conf:{conf}% VSNR:{vsnr} #SuperbetGenesis"
│       │   │   │   │   │   └── return template.format(...)[:MAX_LENGTH]
│       │   │   │   │   └── _truncate_team_name(name: str, max_len: int) -> str
│       │   │   │   │
│       │   │   │   ├── class TelegramFormatter:
│       │   │   │   │   ├── format(prediction: dict) -> str
│       │   │   │   │   │   ├── template = """
│       │   │   │   │   │   ├── 🎯 *{home}* vs *{away}*
│       │   │   │   │   │   ├── 
│       │   │   │   │   │   ├── 🏠 Ev Sahibi: {home_prob}% @{home_odds}
│       │   │   │   │   │   ├── 📊 Güven: {conf}%
│       │   │   │   │   │   ├── 📈 VSNR: {vsnr}
│       │   │   │   │   │   ├── 🎲 CAS: {cas}
│       │   │   │   │   │   ├── 
│       │   │   │   │   │   └── #SuperbetGenesis"""
│       │   │   │   │   └── parse_mode = "Markdown"
│       │   │   │   │
│       │   │   │   └── class AndroidFormatter:
│       │   │   │       ├── format(prediction: dict) -> dict
│       │   │   │       │   └── return {
│       │   │   │       │       ├── "title": f"{home} vs {away}",
│       │   │   │       │       └── "body": f"{pick} {prob}% | Güven: {conf}%"
│       │   │   │       │   }
│       │   │   │       └── MAX_TITLE_LENGTH = 65
│       │   │   │
│       │   │   └── 📄 dispatcher.py
│       │   │       ├── class PriorityDispatcher:
│       │   │       │   ├── """
│       │   │       │   ├── TETRA Panel Kararı:
│       │   │       │   ├──   - max_poll_records=5 (starvation önleme)
│       │   │       │   ├──   - manual commit
│       │   │       │   ├──   - priority header sorting
│       │   │       │   ├── """
│       │   │       │   ├── PLATFORMS = ["twitter", "telegram", "android"]
│       │   │       │   ├── __init__(kafka_producer, topic_prefix="broadcast.outbox")
│       │   │       │   ├── dispatch(event: CloudEvent, priority: float) -> Result
│       │   │       │   │   ├── for platform in PLATFORMS:
│       │   │       │   │   │   ├── topic = f"{topic_prefix}.{platform}"
│       │   │       │   │   │   ├── headers = {"priority": str(priority)}
│       │   │       │   │   │   └── kafka_producer.send(topic, event, headers)
│       │   │       │   │   └── return Result(success=True, platforms=PLATFORMS)
│       │   │       │   └── dispatch_single(event: CloudEvent, platform: str, priority: float)
│       │   │       └── TOPIC_PREFIX = "broadcast.outbox"
│       │   │
│       │   ├── 📁 adapters/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │   ├── from .base import BaseAdapter, Result
│       │   │   │   ├── from .twitter import TwitterAdapter
│       │   │   │   ├── from .telegram import TelegramAdapter
│       │   │   │   └── from .android import AndroidPushAdapter
│       │   │   │
│       │   │   ├── 📄 base.py
│       │   │   │   ├── from abc import ABC, abstractmethod
│       │   │   │   ├── from dataclasses import dataclass
│       │   │   │   ├── from typing import Optional
│       │   │   │   │
│       │   │   │   ├── @dataclass
│       │   │   │   ├── class Result:
│       │   │   │   │   ├── success: bool = False
│       │   │   │   │   ├── retry: bool = False
│       │   │   │   │   ├── reason: Optional[str] = None
│       │   │   │   │   └── error: Optional[Exception] = None
│       │   │   │   │
│       │   │   │   ├── class BaseAdapter(ABC):
│       │   │   │   │   ├── """
│       │   │   │   │   ├── Tüm platform adapter'ları bu interface'i implemente eder.
│       │   │   │   │   ├── SPI (Service Provider Interface) pattern ile dinamik yükleme.
│       │   │   │   │   ├── """
│       │   │   │   │   ├── @abstractmethod
│       │   │   │   │   ├── async def publish(self, cloud_event: dict) -> Result:
│       │   │   │   │   │   └── """Publish CloudEvent to target platform"""
│       │   │   │   │   ├── @abstractmethod
│       │   │   │   │   ├── def get_platform_name(self) -> str:
│       │   │   │   │   │   └── """Return platform identifier"""
│       │   │   │   │   ├── async def publish_with_retry(self, event: dict) -> Result:
│       │   │   │   │   │   ├── """Full Jitter Exponential Backoff ile retry"""
│       │   │   │   │   │   └── # Subclass'ta implement edilir
│       │   │   │   │   └── def get_rate_limit_key(self) -> str:
│       │   │   │   │       └── return f"broadcast:limits:{self.get_platform_name()}"
│       │   │   │   └── ADAPTER_REGISTRY = {}
│       │   │   │
│       │   │   ├── 📄 twitter.py
│       │   │   │   ├── from .base import BaseAdapter, Result
│       │   │   │   ├── from ..resilience import CircuitBreaker, FullJitterBackoff
│       │   │   │   ├── from ..rate_limiting import PlatformRateLimiter
│       │   │   │   │
│       │   │   │   ├── class TwitterAdapter(BaseAdapter):
│       │   │   │   │   ├── """
│       │   │   │   │   ├── X (Twitter) Platform Adapter
│       │   │   │   │   ├── 
│       │   │   │   │   ├── Features:
│       │   │   │   │   ├──   - Token Bucket Rate Limiting
│       │   │   │   │   ├──   - Circuit Breaker (Resilience4j style)
│       │   │   │   │   ├──   - Full Jitter Exponential Backoff
│       │   │   │   │   ├── """
│       │   │   │   │   ├── PLATFORM = "twitter"
│       │   │   │   │   ├── __init__(client: TwitterClient, rate_limiter: PlatformRateLimiter, cb: CircuitBreaker)
│       │   │   │   │   │   ├── self.client = client
│       │   │   │   │   │   ├── self.rate_limiter = rate_limiter
│       │   │   │   │   │   ├── self.cb = cb
│       │   │   │   │   │   └── self.backoff = FullJitterBackoff(base=1.0, cap=60.0, max_attempts=5)
│       │   │   │   │   ├── def get_platform_name(self) -> str:
│       │   │   │   │   │   └── return "twitter"
│       │   │   │   │   ├── async def publish(self, event: dict) -> Result:
│       │   │   │   │   │   ├── # Circuit Breaker check
│       │   │   │   │   │   ├── if self.cb.state == "OPEN":
│       │   │   │   │   │   │   └── return Result(retry=False, reason="cb_open")
│       │   │   │   │   │   ├── # Rate limiter check
│       │   │   │   │   │   ├── if not await self.rate_limiter.try_acquire("twitter", 1):
│       │   │   │   │   │   │   └── return Result(retry=True, reason="rate_limited")
│       │   │   │   │   │   ├── try:
│       │   │   │   │   │   │   ├── text = event["data"]["formatted"]["twitter"]
│       │   │   │   │   │   │   ├── await self.client.tweet(text)
│       │   │   │   │   │   │   ├── self.cb.record_success()
│       │   │   │   │   │   │   └── return Result(success=True)
│       │   │   │   │   │   └── except Exception as e:
│       │   │   │   │   │       ├── self.cb.record_failure()
│       │   │   │   │   │       └── return Result(success=False, error=e)
│       │   │   │   │   └── async def publish_with_retry(self, event: dict) -> Result:
│       │   │   │   │       ├── for attempt in range(self.backoff.max_attempts):
│       │   │   │   │       │   ├── result = await self.publish(event)
│       │   │   │   │       │   ├── if result.success:
│       │   │   │   │       │   │   └── return result
│       │   │   │   │       │   ├── if not result.retry:
│       │   │   │   │       │   │   └── return result
│       │   │   │   │       │   ├── wait_time = await self.backoff.calculate_wait(attempt)
│       │   │   │   │       │   └── await asyncio.sleep(wait_time)
│       │   │   │   │       └── return Result(retry=False, reason="max_attempts_exceeded")
│       │   │   │   │
│       │   │   │   └── class TwitterClient:
│       │   │   │       ├── """Twitter API v2 Client Wrapper"""
│       │   │   │       ├── __init__(api_key, api_secret, access_token, access_secret)
│       │   │   │       ├── async def tweet(text: str) -> TweetResponse
│       │   │   │       ├── async def tweet_with_media(text: str, media_ids: List[str])
│       │   │   │       └── def _handle_rate_limit(response) -> None
│       │   │   │
│       │   │   ├── 📄 telegram.py
│       │   │   │   ├── class TelegramAdapter(BaseAdapter):
│       │   │   │   │   ├── """Telegram Bot API Adapter"""
│       │   │   │   │   ├── PLATFORM = "telegram"
│       │   │   │   │   ├── __init__(bot: TelegramBot, chat_id: str, rate_limiter)
│       │   │   │   │   │   ├── self.bot = bot
│       │   │   │   │   │   ├── self.chat_id = chat_id
│       │   │   │   │   │   └── self.rate_limiter = rate_limiter
│       │   │   │   │   ├── def get_platform_name(self) -> str:
│       │   │   │   │   │   └── return "telegram"
│       │   │   │   │   └── async def publish(self, event: dict) -> Result:
│       │   │   │   │       ├── if not await self.rate_limiter.try_acquire("telegram", 1):
│       │   │   │   │       │   └── return Result(retry=True, reason="rate_limited")
│       │   │   │   │       ├── try:
│       │   │   │   │       │   ├── text = event["data"]["formatted"]["telegram"]
│       │   │   │   │       │   ├── await self.bot.send_message(chat_id, text, parse_mode="Markdown")
│       │   │   │   │       │   └── return Result(success=True)
│       │   │   │   │       └── except Exception as e:
│       │   │   │   │           └── return Result(success=False, error=e)
│       │   │   │   │
│       │   │   │   └── class TelegramBot:
│       │   │   │       ├── """Telegram Bot API Wrapper"""
│       │   │   │       ├── __init__(token: str)
│       │   │   │       ├── async def send_message(chat_id, text, parse_mode)
│       │   │   │       └── async def send_photo(chat_id, photo_url, caption)
│       │   │   │
│       │   │   └── 📄 android.py
│       │   │       ├── class AndroidPushAdapter(BaseAdapter):
│       │   │       │   ├── """Firebase FCM Push Adapter"""
│       │   │       │   ├── PLATFORM = "android"
│       │   │       │   ├── __init__(fcm_client, topic: str, rate_limiter)
│       │   │       │   ├── def get_platform_name(self) -> str:
│       │   │       │   │   └── return "android"
│       │   │       │   └── async def publish(self, event: dict) -> Result:
│       │   │       │       ├── payload = event["data"]["formatted"]["android"]
│       │   │       │       ├── message = Message(
│       │   │       │       │   ├── notification=Notification(
│       │   │       │       │   │   ├── title=payload["title"],
│       │   │       │       │   │   └── body=payload["body"]
│       │   │       │       │   └── ),
│       │   │       │       │   └── topic=self.topic
│       │   │       │       └── )
│       │   │       │       └── await fcm_client.send(message)
│       │   │       │
│       │   │       └── class FCMClient:
│       │   │           ├── """Firebase Cloud Messaging Client"""
│       │   │           ├── __init__(credentials_path: str)
│       │   │           └── async def send(message: Message) -> str
│       │   │
│       │   ├── 📁 rate_limiting/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │
│       │   │   └── 📄 token_bucket.py
│       │   │       ├── import time
│       │   │       ├── import redis
│       │   │       │
│       │   │       ├── class PlatformRateLimiter:
│       │   │       │   ├── """
│       │   │       │   ├── Redis Token Bucket Rate Limiter
│       │   │       │   ├── TETRA Panel Kararı: 8/8 oy
│       │   │       │   ├── Key format: broadcast:limits:{platform}
│       │   │       │   ├── """
│       │   │       │   ├── LIMITS = {
│       │   │       │   │   ├── "twitter": {"daily": 50, "hourly": 10, "refill_rate": 10/3600},
│       │   │       │   │   ├── "telegram": {"daily": 200, "hourly": 30, "refill_rate": 30/3600},
│       │   │       │   │   ├── "android": {"daily": 1000, "hourly": 100, "refill_rate": 100/3600},
│       │   │       │   │   └── "global": {"daily": 200, "hourly": None, "refill_rate": 200/86400}
│       │   │       │   ├── }
│       │   │       │   ├── __init__(redis_client: redis.Redis)
│       │   │       │   │   └── self.redis = redis_client
│       │   │       │   ├── async def try_acquire(platform: str, tokens: int = 1) -> bool
│       │   │       │   │   ├── key = f"broadcast:limits:{platform}"
│       │   │       │   │   ├── limits = self.LIMITS[platform]
│       │   │       │   │   ├── # Lua script for atomic token bucket operation
│       │   │       │   │   ├── script = """
│       │   │       │   │   │   ├── local key = KEYS[1]
│       │   │       │   │   │   ├── local tokens_requested = tonumber(ARGV[1])
│       │   │       │   │   │   ├── local max_tokens = tonumber(ARGV[2])
│       │   │       │   │   │   ├── local refill_rate = tonumber(ARGV[3])
│       │   │       │   │   │   ├── local now = tonumber(ARGV[4])
│       │   │       │   │   │   ├── -- Refill tokens
│       │   │       │   │   │   ├── current_tokens = math.min(max, current + elapsed * rate)
│       │   │       │   │   │   └── return tokens >= requested ? 1 : 0
│       │   │       │   │   └── """
│       │   │       │   │   └── return await self.redis.eval(script, ...) == 1
│       │   │       │   ├── async def get_remaining(platform: str) -> int
│       │   │       │   └── async def reset(platform: str) -> None
│       │   │       │
│       │   │       └── LUA_TOKEN_BUCKET_SCRIPT = """..."""
│       │   │
│       │   ├── 📁 resilience/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │   ├── from .circuit_breaker import CircuitBreaker, CBState, CBConfig
│       │   │   │   ├── from .backoff import FullJitterBackoff
│       │   │   │   └── from .digest_buffer import DigestBuffer
│       │   │   │
│       │   │   ├── 📄 circuit_breaker.py
│       │   │   │   ├── from enum import Enum
│       │   │   │   ├── from dataclasses import dataclass
│       │   │   │   │
│       │   │   │   ├── class CBState(Enum):
│       │   │   │   │   ├── CLOSED = "CLOSED"
│       │   │   │   │   ├── OPEN = "OPEN"
│       │   │   │   │   └── HALF_OPEN = "HALF_OPEN"
│       │   │   │   │
│       │   │   │   ├── @dataclass
│       │   │   │   ├── class CBConfig:
│       │   │   │   │   ├── """
│       │   │   │   │   ├── TETRA Panel Kararı:
│       │   │   │   │   ├──   - failure_threshold: 50%
│       │   │   │   │   ├──   - timeout: 5 dakika (300s)
│       │   │   │   │   ├──   - half_open_calls: 3
│       │   │   │   │   ├── """
│       │   │   │   │   ├── failure_threshold: float = 0.5
│       │   │   │   │   ├── timeout_seconds: int = 300
│       │   │   │   │   └── half_open_calls: int = 3
│       │   │   │   │
│       │   │   │   └── class CircuitBreaker:
│       │   │   │       ├── """Per-adapter Circuit Breaker (TETRA: 8/8 oy)"""
│       │   │   │       ├── __init__(config: CBConfig, digest_buffer: DigestBuffer)
│       │   │   │       │   ├── self.config = config
│       │   │   │       │   ├── self.digest_buffer = digest_buffer
│       │   │   │       │   ├── self.state = CBState.CLOSED
│       │   │   │       │   ├── self.failure_count = 0
│       │   │   │       │   ├── self.success_count = 0
│       │   │   │       │   └── self.total_count = 0
│       │   │   │       ├── @property
│       │   │   │       ├── def failure_rate(self) -> float
│       │   │   │       ├── async def call(adapter: BaseAdapter, event: dict) -> Result
│       │   │   │       │   ├── if self.state == CBState.OPEN:
│       │   │   │       │   │   ├── await self.digest_buffer.buffer_event(event, platform)
│       │   │   │       │   │   └── return Result(retry=False, reason="cb_open")
│       │   │   │       │   └── # Normal flow...
│       │   │   │       ├── def record_success(self) -> None
│       │   │   │       ├── def record_failure(self) -> None
│       │   │   │       ├── def _check_threshold(self) -> None
│       │   │   │       │   ├── if self.failure_rate > self.config.failure_threshold:
│       │   │   │       │   │   ├── self.state = CBState.OPEN
│       │   │   │       │   │   └── asyncio.create_task(self._schedule_half_open())
│       │   │   │       ├── async def _schedule_half_open(self) -> None
│       │   │   │       │   ├── await asyncio.sleep(self.config.timeout_seconds)
│       │   │   │       │   └── self.state = CBState.HALF_OPEN
│       │   │   │       └── def _reset(self) -> None
│       │   │   │
│       │   │   ├── 📄 backoff.py
│       │   │   │   ├── import random
│       │   │   │   ├── import asyncio
│       │   │   │   │
│       │   │   │   └── class FullJitterBackoff:
│       │   │   │       ├── """
│       │   │   │       ├── TETRA Panel Kararı: Full Jitter Exponential Backoff
│       │   │   │       ├── 
│       │   │   │       ├── Formula: wait = min(cap, base * 2**attempt) / 2 + random(0, wait/2)
│       │   │   │       ├── 
│       │   │   │       ├── Parameters:
│       │   │   │       ├──   - base: 1 saniye
│       │   │   │       ├──   - cap: 60 saniye
│       │   │   │       ├──   - max_attempts: 5
│       │   │   │       ├── 
│       │   │   │       ├── Gerekçe: Thundering herd'ü önler
│       │   │   │       ├── """
│       │   │   │       ├── def __init__(base: float = 1.0, cap: float = 60.0, max_attempts: int = 5)
│       │   │   │       │   ├── self.base = base
│       │   │   │       │   ├── self.cap = cap
│       │   │   │       │   └── self.max_attempts = max_attempts
│       │   │   │       └── async def calculate_wait(attempt: int) -> float
│       │   │   │           ├── inner = min(self.cap, self.base * (2 ** attempt))
│       │   │   │           └── return inner / 2 + random.uniform(0, inner / 2)
│       │   │   │
│       │   │   └── 📄 digest_buffer.py
│       │   │       ├── import json
│       │   │       ├── import time
│       │   │       │
│       │   │       └── class DigestBuffer:
│       │   │           ├── """
│       │   │           ├── TETRA Panel Kararı: DLQ yerine Digest Buffer (7/8 oy)
│       │   │           ├── 
│       │   │           ├── Mekanizma:
│       │   │           ├──   CB OPEN → Event buffer'a düşer
│       │   │           ├──   15dk bekle → Batch retry
│       │   │           ├──   Başarısız → DLQ (final)
│       │   │           ├── 
│       │   │           ├── Key format: broadcast:digest:{platform}:{bucket_id}
│       │   │           ├── bucket_id = int(timestamp // 900)  # 15dk bucket
│       │   │           ├── """
│       │   │           ├── TTL_SECONDS = 900  # 15 dakika
│       │   │           ├── __init__(redis_client, kafka_producer)
│       │   │           │   ├── self.redis = redis_client
│       │   │           │   └── self.kafka = kafka_producer
│       │   │           ├── def _get_bucket_key(platform: str) -> str
│       │   │           │   ├── bucket_id = int(time.time() // self.TTL_SECONDS)
│       │   │           │   └── return f"broadcast:digest:{platform}:{bucket_id}"
│       │   │           ├── async def buffer_event(event: dict, platform: str) -> None
│       │   │           │   ├── key = self._get_bucket_key(platform)
│       │   │           │   ├── await self.redis.rpush(key, json.dumps(event))
│       │   │           │   └── await self.redis.expire(key, self.TTL_SECONDS * 2)
│       │   │           ├── async def flush_bucket(platform: str, bucket_key: str, adapter) -> dict
│       │   │           │   ├── events = await self.redis.lrange(bucket_key, 0, -1)
│       │   │           │   ├── results = {"success": 0, "failed": 0}
│       │   │           │   ├── for event_json in events:
│       │   │           │   │   ├── event = json.loads(event_json)
│       │   │           │   │   ├── result = await adapter.publish_with_retry(event)
│       │   │           │   │   ├── if result.success:
│       │   │           │   │   │   └── results["success"] += 1
│       │   │           │   │   └── else:
│       │   │           │   │       ├── results["failed"] += 1
│       │   │           │   │       └── await self._send_to_dlq(event, platform, result.reason)
│       │   │           │   ├── await self.redis.delete(bucket_key)
│       │   │           │   └── return results
│       │   │           └── async def _send_to_dlq(event, platform, reason)
│       │   │               └── await self.kafka.send("broadcast.dlq", {...})
│       │   │
│       │   ├── 📁 kafka/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │
│       │   │   ├── 📄 consumer.py
│       │   │   │   ├── class PriorityConsumer:
│       │   │   │   │   ├── """
│       │   │   │   │   ├── TETRA Panel Kararı:
│       │   │   │   │   ├──   - enable.auto.commit=False (manual commit)
│       │   │   │   │   ├──   - max_poll_records=5 (starvation önleme)
│       │   │   │   │   ├── """
│       │   │   │   │   ├── CONFIG = {
│       │   │   │   │   │   ├── "bootstrap.servers": "kafka:9092",
│       │   │   │   │   │   ├── "group.id": "broadcast-plant-priority",
│       │   │   │   │   │   ├── "auto.offset.reset": "earliest",
│       │   │   │   │   │   ├── "enable.auto.commit": False,
│       │   │   │   │   │   ├── "max.poll.records": 5,
│       │   │   │   │   │   ├── "fetch.min.bytes": 1024,
│       │   │   │   │   │   └── "fetch.max.wait.ms": 500
│       │   │   │   │   ├── }
│       │   │   │   │   ├── __init__(topic: str, config: dict = None)
│       │   │   │   │   ├── async def consume() -> List[Message]
│       │   │   │   │   │   ├── messages = await self.consumer.poll(timeout_ms=1000)
│       │   │   │   │   │   ├── # Priority'ye göre sırala
│       │   │   │   │   │   ├── prioritized = []
│       │   │   │   │   │   ├── for tp, records in messages.items():
│       │   │   │   │   │   │   ├── for record in records:
│       │   │   │   │   │   │   │   ├── priority = float(record.headers.get("priority", 0.0))
│       │   │   │   │   │   │   │   └── prioritized.append((priority, record))
│       │   │   │   │   │   ├── prioritized.sort(key=lambda x: -x[0])  # Yüksek önce
│       │   │   │   │   │   └── return [r for _, r in prioritized[:5]]
│       │   │   │   │   └── async def commit() -> None
│       │   │   │   └── CONSUMER_GROUP = "broadcast-plant-priority"
│       │   │   │
│       │   │   ├── 📄 producer.py
│       │   │   │   └── class BroadcastProducer:
│       │   │   │       ├── __init__(bootstrap_servers: str)
│       │   │   │       ├── async def send(topic: str, event: dict, headers: dict = None)
│       │   │   │       │   ├── key = event["id"]
│       │   │   │       │   ├── value = json.dumps(event).encode()
│       │   │   │       │   └── await self.producer.send(topic, key, value, headers)
│       │   │   │       └── async def flush() -> None
│       │   │   │
│       │   │   └── 📄 topics.py
│       │   │       ├── """
│       │   │       ├── TETRA Panel Kararı: Per-platform outbox topics (7/8 oy)
│       │   │       ├── Bulkhead Pattern: X API yavaşlarsa Telegram etkilenmez
│       │   │       ├── """
│       │   │       ├── TOPICS = {
│       │   │       │   ├── "input": "risk.verified",
│       │   │       │   ├── "priority_queue": "broadcast.queue.priority",
│       │   │       │   ├── "outbox_twitter": "broadcast.outbox.twitter",
│       │   │       │   ├── "outbox_telegram": "broadcast.outbox.telegram",
│       │   │       │   ├── "outbox_android": "broadcast.outbox.android",
│       │   │       │   ├── "digest_buffer": "broadcast.digest.buffer",
│       │   │       │   └── "dlq": "broadcast.dlq"
│       │   │       ├── }
│       │   │       └── CONSUMER_GROUPS = {
│       │   │           ├── "main": "broadcast-plant-main",
│       │   │           ├── "priority": "broadcast-plant-priority",
│       │   │           ├── "twitter": "twitter-adapter-consumer",
│       │   │           ├── "telegram": "telegram-adapter-consumer",
│       │   │           └── "android": "android-adapter-consumer"
│       │   │       }
│       │   │
│       │   ├── 📁 monitoring/
│       │   │   │
│       │   │   ├── 📄 __init__.py
│       │   │   │
│       │   │   ├── 📄 metrics.py
│       │   │   │   ├── from prometheus_client import Counter, Gauge, Histogram
│       │   │   │   │
│       │   │   │   ├── # Counters
│       │   │   │   ├── broadcast_published_total = Counter(
│       │   │   │   │   ├── "broadcast_published_total",
│       │   │   │   │   ├── "Total published broadcasts",
│       │   │   │   │   └── ["platform", "priority"]
│       │   │   │   ├── )
│       │   │   │   ├── broadcast_dropped_total = Counter(
│       │   │   │   │   ├── "broadcast_dropped_total",
│       │   │   │   │   ├── "Total dropped broadcasts",
│       │   │   │   │   └── ["reason"]
│       │   │   │   ├── )
│       │   │   │   │
│       │   │   │   ├── # Gauges
│       │   │   │   ├── broadcast_circuit_breaker_state = Gauge(
│       │   │   │   │   ├── "broadcast_circuit_breaker_state",
│       │   │   │   │   ├── "Circuit breaker state (0=CLOSED, 1=OPEN, 2=HALF_OPEN)",
│       │   │   │   │   └── ["platform"]
│       │   │   │   ├── )
│       │   │   │   ├── broadcast_digest_buffer_size = Gauge(
│       │   │   │   │   ├── "broadcast_digest_buffer_size",
│       │   │   │   │   ├── "Number of events in digest buffer",
│       │   │   │   │   └── ["platform"]
│       │   │   │   ├── )
│       │   │   │   ├── broadcast_digest_staleness_seconds = Gauge(
│       │   │   │   │   ├── "broadcast_digest_staleness_seconds",
│       │   │   │   │   ├── "Age of oldest event in digest buffer",
│       │   │   │   │   └── ["platform"]
│       │   │   │   ├── )
│       │   │   │   │
│       │   │   │   └── # Histograms
│       │   │   │       └── broadcast_latency_seconds = Histogram(
│       │   │   │           ├── "broadcast_latency_seconds",
│       │   │   │           ├── "Broadcast latency",
│       │   │   │           ├── ["platform"],
│       │   │   │           └── buckets=[0.01, 0.05, 0.1, 0.5, 1.0, 5.0]
│       │   │   │       )
│       │   │   │
│       │   │   └── 📄 alerting.py
│       │   │       ├── """
│       │   │       ├── Prometheus Alert Rules (TETRA Panel)
│       │   │       ├── """
│       │   │       ├── ALERT_RULES = {
│       │   │       │   ├── "DigestBufferStale": {
│       │   │       │   │   ├── "expr": "broadcast_digest_staleness_seconds > 900",
│       │   │       │   │   ├── "for": "1m",
│       │   │       │   │   ├── "severity": "critical",
│       │   │       │   │   └── "summary": "Digest Buffer stale"
│       │   │       │   ├── },
│       │   │       │   ├── "CircuitBreakerOpen": {
│       │   │       │   │   ├── "expr": "broadcast_circuit_breaker_state == 1",
│       │   │       │   │   ├── "for": "5m",
│       │   │       │   │   └── "severity": "warning"
│       │   │       │   ├── },
│       │   │       │   └── "HighErrorRate": {
│       │   │       │       ├── "expr": "broadcast_error_rate > 0.1",
│       │   │       │       └── "severity": "warning"
│       │   │       │   }
│       │   │       └── }
│       │   │
│       │   └── 📁 schemas/
│       │       │
│       │       ├── 📄 cloud_event.proto
│       │       │   ├── message CloudEvent {
│       │       │   │   ├── string specversion = 1;  // "1.0"
│       │       │   │   ├── string type = 2;         // "com.superbet.prediction.published"
│       │       │   │   ├── string source = 3;       // "/plant/broadcast"
│       │       │   │   ├── string id = 4;           // "pred-{match_id}-{timestamp}"
│       │       │   │   ├── string datacontenttype = 5;
│       │       │   │   ├── string time = 6;
│       │       │   │   └── BroadcastData data = 7;
│       │       │   └── }
│       │       │
│       │       ├── 📄 broadcast_data.proto
│       │       │   ├── message BroadcastData {
│       │       │   │   ├── string match_id = 1;
│       │       │   │   ├── Teams teams = 2;
│       │       │   │   ├── Prediction prediction = 3;
│       │       │   │   ├── Odds odds = 4;
│       │       │   │   ├── Metrics metrics = 5;
│       │       │   │   ├── FormattedMessages formatted = 6;
│       │       │   │   └── Metadata metadata = 7;
│       │       │   ├── }
│       │       │   ├── message Teams {
│       │       │   │   ├── string home = 1;
│       │       │   │   └── string away = 2;
│       │       │   ├── }
│       │       │   ├── message Prediction {
│       │       │   │   ├── string pick = 1;
│       │       │   │   ├── float home_win = 2;
│       │       │   │   ├── float draw = 3;
│       │       │   │   └── float away_win = 4;
│       │       │   ├── }
│       │       │   ├── message Metrics {
│       │       │   │   ├── float confidence = 1;
│       │       │   │   ├── float vsnr = 2;
│       │       │   │   ├── float cas = 3;
│       │       │   │   ├── float kelly_fraction = 4;
│       │       │   │   └── float gamma = 5;
│       │       │   ├── }
│       │       │   └── message FormattedMessages {
│       │       │       ├── string twitter = 1;
│       │       │       ├── string telegram = 2;
│       │       │       └── AndroidPush android = 3;
│       │       │   }
│       │       │
│       │       └── 📄 priority_header.proto
│       │           └── message PriorityHeader {
│       │               ├── float score = 1;
│       │               ├── string tier = 2;  // critical, high, medium, low
│       │               └── int64 timestamp = 3;
│       │           }
│       │
│       ├── 📁 tests/
│       │   ├── 📁 unit/
│       │   │   ├── 📄 test_filter.py
│       │   │   │   ├── def test_should_broadcast_passes_all_thresholds()
│       │   │   │   ├── def test_should_broadcast_fails_low_confidence()
│       │   │   │   ├── def test_should_broadcast_fails_low_vsnr()
│       │   │   │   └── def test_should_broadcast_fails_low_cas()
│       │   │   ├── 📄 test_priority_scorer.py
│       │   │   │   ├── def test_priority_score_calculation()
│       │   │   │   ├── def test_vsnr_clipping()
│       │   │   │   └── def test_cas_clipping()
│       │   │   ├── 📄 test_circuit_breaker.py
│       │   │   │   ├── def test_cb_opens_on_threshold()
│       │   │   │   ├── def test_cb_half_open_after_timeout()
│       │   │   │   └── def test_cb_closes_after_success()
│       │   │   ├── 📄 test_backoff.py
│       │   │   │   ├── def test_full_jitter_bounds()
│       │   │   │   └── def test_exponential_growth()
│       │   │   └── 📄 test_token_bucket.py
│       │   │       ├── def test_acquire_within_limit()
│       │   │       └── def test_acquire_exceeds_limit()
│       │   │
│       │   ├── 📁 integration/
│       │   │   ├── 📄 test_twitter_adapter.py
│       │   │   ├── 📄 test_kafka_consumer.py
│       │   │   └── 📄 test_redis_rate_limiter.py
│       │   │
│       │   └── 📁 fixtures/
│       │       ├── 📄 sample_predictions.json
│       │       └── 📄 sample_cloud_events.json
│       │
│       ├── 📄 pyproject.toml
│       ├── 📄 Dockerfile
│       ├── 📄 docker-compose.yml   # Local development
│       └── 📄 README.md
```

---

## 📁 Helm Chart Eklentisi (infra/helm/charts/)

```
infra/helm/charts/
│
└── 📁 broadcast-plant/
    ├── 📄 Chart.yaml
    │   ├── name: broadcast-plant
    │   ├── version: 1.0.0
    │   ├── appVersion: v3.1
    │   └── dependencies:
    │       └── redis: ">=6.0.0"
    │
    ├── 📄 values.yaml
    │   ├── replicaCount: 2
    │   ├── image:
    │   │   ├── repository: superbet/broadcast-plant
    │   │   └── tag: v3.1
    │   ├── kafka:
    │   │   ├── bootstrapServers: kafka:9092
    │   │   └── topics:
    │   │       ├── input: risk.verified
    │   │       ├── outbox_twitter: broadcast.outbox.twitter
    │   │       ├── outbox_telegram: broadcast.outbox.telegram
    │   │       ├── outbox_android: broadcast.outbox.android
    │   │       └── dlq: broadcast.dlq
    │   ├── redis:
    │   │   └── host: redis-cluster:6379
    │   ├── adapters:
    │   │   ├── twitter:
    │   │   │   ├── enabled: true
    │   │   │   └── rateLimits:
    │   │   │       ├── daily: 50
    │   │   │       └── hourly: 10
    │   │   ├── telegram:
    │   │   │   ├── enabled: true
    │   │   │   └── rateLimits:
    │   │   │       ├── daily: 200
    │   │   │       └── hourly: 30
    │   │   └── android:
    │   │       ├── enabled: false  # Phase 2
    │   │       └── rateLimits:
    │   │           ├── daily: 1000
    │   │           └── hourly: 100
    │   ├── filter:
    │   │   ├── confidence: 0.65
    │   │   ├── vsnr: 2.2
    │   │   └── cas: 1.0
    │   ├── circuitBreaker:
    │   │   ├── failureThreshold: 0.5
    │   │   ├── timeoutSeconds: 300
    │   │   └── halfOpenCalls: 3
    │   ├── digestBuffer:
    │   │   └── ttlSeconds: 900
    │   └── resources:
    │       ├── requests:
    │       │   ├── cpu: 100m
    │       │   └── memory: 256Mi
    │       └── limits:
    │           ├── cpu: 500m
    │           └── memory: 512Mi
    │
    ├── 📁 templates/
    │   ├── 📄 deployment.yaml
    │   ├── 📄 service.yaml
    │   ├── 📄 configmap.yaml
    │   ├── 📄 secret.yaml           # Twitter/Telegram API keys
    │   ├── 📄 hpa.yaml              # HorizontalPodAutoscaler
    │   ├── 📄 servicemonitor.yaml   # Prometheus ServiceMonitor
    │   └── 📄 prometheusrule.yaml   # Alert rules
    │
    └── 📄 README.md
```

---

## 📁 Kafka Topic Konfigürasyonu

```
infra/kafka/topics/
│
├── 📄 broadcast-topics.yaml
│   ├── apiVersion: kafka.strimzi.io/v1beta2
│   ├── kind: KafkaTopic
│   ├── metadata:
│   │   └── name: broadcast.outbox.twitter
│   ├── spec:
│   │   ├── partitions: 3
│   │   ├── replicas: 2
│   │   └── config:
│   │       ├── retention.ms: 604800000  # 7 days
│   │       └── cleanup.policy: delete
│   │
│   ├── # Aynı yapıda:
│   ├── # - broadcast.outbox.telegram
│   ├── # - broadcast.outbox.android
│   ├── # - broadcast.queue.priority
│   ├── # - broadcast.digest.buffer
│   └── # - broadcast.dlq
│
└── 📄 consumer-groups.yaml
    ├── groups:
    │   ├── broadcast-plant-main
    │   ├── broadcast-plant-priority
    │   ├── twitter-adapter-consumer
    │   ├── telegram-adapter-consumer
    │   └── android-adapter-consumer
```

---

## DEVAMI → PROJECT_TREE_v3.1_PART6.md (Monitoring Dashboard) veya bağımsız kullanım
