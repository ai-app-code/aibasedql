# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 2
## Services + ML Katmanı (4+ Seviye Derinlik)

---

```
superbet-genesis/ (devam)
│
├── 📁 services/
│   │
│   ├── 📁 data-plant/
│   │   ├── 📁 src/
│   │   │   ├── 📁 adapters/
│   │   │   │   ├── 📁 api_football/
│   │   │   │   │   ├── 📄 __init__.py
│   │   │   │   │   ├── 📄 client.py
│   │   │   │   │   │   ├── class APIFootballClient:
│   │   │   │   │   │   │   ├── __init__(api_key, base_url, rate_limiter)
│   │   │   │   │   │   │   ├── get_fixtures(league_id, season, date_range)
│   │   │   │   │   │   │   ├── get_live_matches()
│   │   │   │   │   │   │   ├── get_odds(fixture_id, bookmaker)
│   │   │   │   │   │   │   ├── get_statistics(fixture_id)
│   │   │   │   │   │   │   ├── get_lineups(fixture_id)
│   │   │   │   │   │   │   ├── get_events(fixture_id)
│   │   │   │   │   │   │   └── _handle_rate_limit(response)
│   │   │   │   │   │   └── class AsyncAPIFootballClient: [async variants]
│   │   │   │   │   │
│   │   │   │   │   ├── 📄 rate_limiter.py
│   │   │   │   │   │   ├── class RedisTokenBucket:
│   │   │   │   │   │   │   ├── __init__(redis_client, bucket_size=100, refill_rate=10)
│   │   │   │   │   │   │   ├── acquire(tokens=1) -> bool
│   │   │   │   │   │   │   ├── _lua_script: [atomik token kontrolü]
│   │   │   │   │   │   │   └── get_remaining_tokens() -> int
│   │   │   │   │   │   └── class SlidingWindowLimiter: [alternatif]
│   │   │   │   │   │
│   │   │   │   │   ├── 📄 response_parser.py
│   │   │   │   │   │   ├── class FixtureParser:
│   │   │   │   │   │   │   ├── parse_fixture(raw_json) -> CanonicalMatch
│   │   │   │   │   │   │   ├── parse_statistics(raw_json) -> MatchStatistics
│   │   │   │   │   │   │   └── _normalize_team_name(name) -> str
│   │   │   │   │   │   ├── class OddsParser:
│   │   │   │   │   │   │   ├── parse_odds(raw_json) -> OddsSnapshot
│   │   │   │   │   │   │   └── _calculate_implied_probability(odds)
│   │   │   │   │   │   └── class EventParser: [gol, kart, değişiklik]
│   │   │   │   │   │
│   │   │   │   │   └── 📄 endpoints.py
│   │   │   │   │       ├── FIXTURES = "/fixtures"
│   │   │   │   │       ├── ODDS = "/odds"
│   │   │   │   │       ├── STATISTICS = "/fixtures/statistics"
│   │   │   │   │       └── LINEUPS = "/fixtures/lineups"
│   │   │   │   │
│   │   │   │   └── 📁 external/
│   │   │   │       ├── 📄 weather_api.py
│   │   │   │       │   └── class WeatherClient:
│   │   │   │       │       ├── get_match_weather(lat, lon, kickoff_time)
│   │   │   │       │       └── _parse_conditions() -> WeatherData
│   │   │   │       └── 📄 news_scraper.py
│   │   │   │           └── class NewsSentimentScraper:
│   │   │   │               ├── scrape_team_news(team_id)
│   │   │   │               └── analyze_sentiment(text) -> float [-1, 1]
│   │   │   │
│   │   │   ├── 📁 ingestion/
│   │   │   │   ├── 📄 conflict_resolver.py
│   │   │   │   │   ├── class ConflictResolver:
│   │   │   │   │   │   ├── __init__(primary_source, fallback_sources)
│   │   │   │   │   │   ├── resolve(data_point) -> ResolvedData
│   │   │   │   │   │   │   ├── _check_monotonicity(old, new) -> bool
│   │   │   │   │   │   │   ├── _merge_partial_updates(sources)
│   │   │   │   │   │   │   └── _select_winner(candidates) -> Data
│   │   │   │   │   │   └── failover_to_secondary() -> bool
│   │   │   │   │   └── enum ConflictStrategy: [LATEST_WINS, MAJORITY_VOTE, WEIGHTED]
│   │   │   │   │
│   │   │   │   ├── 📄 coverage_manager.py
│   │   │   │   │   ├── class CoverageManager:
│   │   │   │   │   │   ├── check_coverage(match_data) -> CoverageReport
│   │   │   │   │   │   │   ├── required_fields: [home_team, away_team, kickoff, odds]
│   │   │   │   │   │   │   ├── optional_fields: [xg, possession, corners]
│   │   │   │   │   │   │   └── coverage_score: float [0-1]
│   │   │   │   │   │   ├── impute_missing(data, strategy) -> ImputedData
│   │   │   │   │   │   │   ├── strategies: [MEAN, MEDIAN, KNN, MODEL_BASED]
│   │   │   │   │   │   │   └── confidence_penalty: [0.1 per missing field]
│   │   │   │   │   │   └── calculate_confidence(data) -> float
│   │   │   │   │   └── COVERAGE_THRESHOLD = 0.95
│   │   │   │   │
│   │   │   │   ├── 📄 freshness_scorer.py
│   │   │   │   │   ├── class FreshnessScorer:
│   │   │   │   │   │   ├── calculate_freshness(data_timestamp, now) -> float
│   │   │   │   │   │   │   ├── formula: exp(-λ * (now - timestamp))
│   │   │   │   │   │   │   └── λ = 0.1 (decay rate)
│   │   │   │   │   │   ├── should_skip(freshness) -> bool
│   │   │   │   │   │   │   └── threshold: 0.3
│   │   │   │   │   │   └── get_staleness_bucket(freshness) -> str
│   │   │   │   │   │       ├── "fresh": > 0.8
│   │   │   │   │   │       ├── "acceptable": 0.5-0.8
│   │   │   │   │   │       ├── "stale": 0.3-0.5
│   │   │   │   │   │       └── "skip": < 0.3
│   │   │   │   │   └── FRESHNESS_GATE = 30  # seconds
│   │   │   │   │
│   │   │   │   └── 📄 monotonicity_check.py
│   │   │   │       └── class MonotonicityValidator:
│   │   │   │           ├── validate_score(old, new) -> bool [score never decreases]
│   │   │   │           ├── validate_minute(old, new) -> bool [minute only increases]
│   │   │   │           └── validate_odds(old, new) -> OddsChange
│   │   │   │
│   │   │   ├── 📁 producers/
│   │   │   │   ├── 📄 match_producer.py
│   │   │   │   │   ├── class MatchUpdateProducer:
│   │   │   │   │   │   ├── __init__(kafka_config, topic="football.match.update")
│   │   │   │   │   │   ├── produce(match: CanonicalMatch)
│   │   │   │   │   │   │   ├── serialize: protobuf
│   │   │   │   │   │   │   ├── key: match_id
│   │   │   │   │   │   │   └── headers: [source, timestamp, version]
│   │   │   │   │   │   ├── produce_batch(matches: List[CanonicalMatch])
│   │   │   │   │   │   └── _add_cloud_event_envelope(data) -> CloudEvent
│   │   │   │   │   └── SCHEMA_VERSION = "1.0.0"
│   │   │   │   │
│   │   │   │   ├── 📄 odds_producer.py
│   │   │   │   │   └── class OddsUpdateProducer:
│   │   │   │   │       ├── produce(odds: OddsSnapshot)
│   │   │   │   │       ├── topic: "football.odds.update"
│   │   │   │   │       └── partition_key: fixture_id
│   │   │   │   │
│   │   │   │   ├── 📄 live_producer.py
│   │   │   │   │   └── class LiveEventProducer:
│   │   │   │   │       ├── produce_goal(fixture_id, minute, team, player)
│   │   │   │   │       ├── produce_card(fixture_id, minute, card_type, player)
│   │   │   │   │       ├── produce_substitution(fixture_id, minute, player_in, player_out)
│   │   │   │   │       └── topic: "football.live.events"
│   │   │   │   │
│   │   │   │   └── 📄 cloud_events.py
│   │   │   │       ├── class CloudEventBuilder:
│   │   │   │       │   ├── spec_version: "1.0"
│   │   │   │       │   ├── type: "com.superbet.football.{event_type}"
│   │   │   │       │   ├── source: "/data-plant/api-football"
│   │   │   │       │   └── build(data, event_type) -> CloudEvent
│   │   │   │       └── CONTENT_TYPE = "application/protobuf"
│   │   │   │
│   │   │   ├── 📁 schemas/
│   │   │   │   ├── 📄 match.proto
│   │   │   │   │   ├── message CanonicalMatch:
│   │   │   │   │   │   ├── int64 match_id
│   │   │   │   │   │   ├── int32 league_id
│   │   │   │   │   │   ├── Team home_team
│   │   │   │   │   │   ├── Team away_team
│   │   │   │   │   │   ├── int64 kickoff_timestamp
│   │   │   │   │   │   ├── MatchStatus status
│   │   │   │   │   │   ├── Score score
│   │   │   │   │   │   ├── int32 minute
│   │   │   │   │   │   └── float confidence
│   │   │   │   │   ├── message Team: [id, name, logo_url]
│   │   │   │   │   ├── message Score: [home, away]
│   │   │   │   │   └── enum MatchStatus: [SCHEDULED, LIVE, HALFTIME, FINISHED, POSTPONED]
│   │   │   │   │
│   │   │   │   ├── 📄 odds.proto
│   │   │   │   │   ├── message OddsSnapshot:
│   │   │   │   │   │   ├── int64 fixture_id
│   │   │   │   │   │   ├── int64 timestamp
│   │   │   │   │   │   ├── repeated MarketOdds markets
│   │   │   │   │   │   └── string bookmaker
│   │   │   │   │   └── message MarketOdds:
│   │   │   │   │       ├── string market_type [1X2, BTTS, O/U]
│   │   │   │   │       └── repeated Selection selections
│   │   │   │   │
│   │   │   │   ├── 📄 twin_delta.proto
│   │   │   │   │   └── message TwinDelta:
│   │   │   │   │       ├── int64 version
│   │   │   │   │       ├── float home_xg, away_xg
│   │   │   │   │       ├── float home_possession, away_possession
│   │   │   │   │       ├── int32 home_score, away_score
│   │   │   │   │       ├── int32 minute
│   │   │   │   │       ├── bytes graph_blob [GNN embedding]
│   │   │   │   │       └── uint32 crc32 [integrity check]
│   │   │   │   │
│   │   │   │   └── 📄 graph_blob.proto
│   │   │   │       └── message GraphEmbedding:
│   │   │   │           ├── repeated float team_embedding [128 dim]
│   │   │   │           ├── repeated float match_embedding [128 dim]
│   │   │   │           └── int64 timestamp
│   │   │   │
│   │   │   └── 📁 quality/
│   │   │       ├── 📄 expectations.py
│   │   │       │   ├── class MatchDataValidator:
│   │   │       │   │   ├── expect_column_values_to_not_be_null(["match_id", "kickoff"])
│   │   │       │   │   ├── expect_column_values_to_be_between("minute", 0, 120)
│   │   │       │   │   ├── expect_column_values_to_be_in_set("status", [valid_statuses])
│   │   │       │   │   └── expect_compound_columns_to_be_unique(["match_id", "timestamp"])
│   │   │       │   └── class OddsDataValidator: [odds > 1.0, margin < 15%]
│   │   │       │
│   │   │       ├── 📄 freshness_gate.py
│   │   │       │   └── class FreshnessGate:
│   │   │       │       ├── check(data) -> GateResult
│   │   │       │       │   ├── max_age: 30 seconds
│   │   │       │       │   ├── pass: data_age < max_age
│   │   │       │       │   └── fail_action: SKIP or ALERT
│   │   │       │       └── get_metrics() -> dict [pass_rate, avg_age]
│   │   │       │
│   │   │       └── 📄 coverage_gate.py
│   │   │           └── class CoverageGate:
│   │   │               ├── check(data) -> GateResult
│   │   │               │   ├── min_coverage: 95%
│   │   │               │   └── required_fields: [match_id, odds, kickoff]
│   │   │               └── impute_strategy: KNN
│   │   │
│   │   ├── 📁 tests/
│   │   │   ├── 📁 unit/
│   │   │   │   ├── 📄 test_rate_limiter.py
│   │   │   │   ├── 📄 test_conflict_resolver.py
│   │   │   │   ├── 📄 test_freshness_scorer.py
│   │   │   │   └── 📄 test_coverage_manager.py
│   │   │   ├── 📁 integration/
│   │   │   │   ├── 📄 test_api_football_client.py
│   │   │   │   └── 📄 test_kafka_producer.py
│   │   │   └── 📁 fixtures/
│   │   │       ├── 📄 sample_fixtures.json
│   │   │       └── 📄 sample_odds.json
│   │   │
│   │   ├── 📄 pyproject.toml
│   │   ├── 📄 Dockerfile
│   │   └── 📄 README.md
│   │
│   ├── 📁 stream-processor/
│   │   ├── 📁 src/main/java/com/superbet/stream/
│   │   │   ├── 📁 jobs/
│   │   │   │   ├── 📄 MatchUpdateJob.java
│   │   │   │   │   ├── class MatchUpdateJob extends FlinkJob:
│   │   │   │   │   │   ├── source: KafkaSource("football.match.update")
│   │   │   │   │   │   ├── operators:
│   │   │   │   │   │   │   ├── DeduplicationOperator(window=5min)
│   │   │   │   │   │   │   ├── EnrichmentOperator(feature_store)
│   │   │   │   │   │   │   └── AggregationOperator(tumbling=1min)
│   │   │   │   │   │   └── sinks:
│   │   │   │   │   │       ├── ClickHouseSink(table="matches")
│   │   │   │   │   │       ├── RedisSink(prefix="match:")
│   │   │   │   │   │       └── DeltaLakeSink(path="s3://data/matches/")
│   │   │   │   │   └── watermark: EventTimeWatermark(maxOutOfOrderness=10s)
│   │   │   │   │
│   │   │   │   ├── 📄 OddsStreamJob.java
│   │   │   │   │   └── class OddsStreamJob:
│   │   │   │   │       ├── source: KafkaSource("football.odds.update")
│   │   │   │   │       ├── operators:
│   │   │   │   │       │   ├── OddsChangeDetector(threshold=0.05)
│   │   │   │   │       │   ├── ImpliedProbabilityCalculator
│   │   │   │   │       │   └── SignificantMoveAlert(threshold=0.1)
│   │   │   │   │       └── sinks: [ClickHouse, Redis, Feast]
│   │   │   │   │
│   │   │   │   ├── 📄 FeatureEnrichmentJob.java
│   │   │   │   │   └── class FeatureEnrichmentJob:
│   │   │   │   │       ├── async_io: FeastAsyncLookup
│   │   │   │   │       ├── enrichment: [team_form, h2h, player_ratings]
│   │   │   │   │       └── output: EnrichedMatchEvent
│   │   │   │   │
│   │   │   │   └── 📄 GraphEventJob.java
│   │   │   │       └── class GraphEventJob:
│   │   │   │           ├── source: KafkaSource("graph.events")
│   │   │   │           ├── neo4j_sink: Neo4jBatchWriter
│   │   │   │           └── milvus_sink: MilvusEmbeddingWriter
│   │   │   │
│   │   │   ├── 📁 operators/
│   │   │   │   ├── 📄 DeduplicationOperator.java
│   │   │   │   │   └── class DeduplicationOperator:
│   │   │   │   │       ├── state: MapState<String, Long> [event_id -> timestamp]
│   │   │   │   │       ├── window: 5 minutes
│   │   │   │   │       └── isDuplicate(event) -> boolean
│   │   │   │   │
│   │   │   │   ├── 📄 WindowAggregator.java
│   │   │   │   │   └── class WindowAggregator:
│   │   │   │   │       ├── windows: [tumbling_1min, sliding_5min, session_30min]
│   │   │   │   │       └── aggregations: [count, sum, avg, min, max]
│   │   │   │   │
│   │   │   │   └── 📄 WatermarkAssigner.java
│   │   │   │       └── class BoundedOutOfOrdernessWatermarkAssigner:
│   │   │   │           ├── maxOutOfOrderness: 10 seconds
│   │   │   │           └── extractTimestamp(event) -> long
│   │   │   │
│   │   │   └── 📁 sinks/
│   │   │       ├── 📄 ClickHouseSink.java
│   │   │       │   └── class ClickHouseSink:
│   │   │       │       ├── batch_size: 1000
│   │   │       │       ├── flush_interval: 1 second
│   │   │       │       └── insert_query: "INSERT INTO {table} FORMAT Native"
│   │   │       │
│   │   │       ├── 📄 RedisSink.java
│   │   │       │   └── class RedisSink:
│   │   │       │       ├── connection_pool: 10
│   │   │       │       ├── ttl: 30 seconds
│   │   │       │       └── lua_cas_script: [version check before write]
│   │   │       │
│   │   │       ├── 📄 DeltaLakeSink.java
│   │   │       │   └── class DeltaLakeSink:
│   │   │       │       ├── mode: MergeOnRead (Hudi)
│   │   │       │       ├── partition_by: [date, league_id]
│   │   │       │       └── compaction: async, every 1 hour
│   │   │       │
│   │   │       └── 📄 FeastSink.java
│   │   │           └── class FeastSink:
│   │   │               ├── feature_views: [match_stats, live_odds]
│   │   │               └── push_mode: online_only
│   │   │
│   │   ├── 📄 pom.xml
│   │   ├── 📄 Dockerfile
│   │   └── 📄 README.md
│   │
│   ├── 📁 feature-store/
│   │   ├── 📁 feature_repo/
│   │   │   ├── 📄 feature_store.yaml
│   │   │   │   ├── project: superbet
│   │   │   │   ├── registry: s3://feast/registry.db
│   │   │   │   ├── provider: aws
│   │   │   │   ├── online_store:
│   │   │   │   │   ├── type: redis
│   │   │   │   │   ├── connection_string: redis-cluster:6379
│   │   │   │   │   └── key_ttl_seconds: 86400
│   │   │   │   └── offline_store:
│   │   │   │       ├── type: file  # or spark, bigquery
│   │   │   │       └── path: s3://feast/offline/
│   │   │   │
│   │   │   ├── 📁 entities/
│   │   │   │   ├── 📄 match.py
│   │   │   │   │   └── match = Entity(name="match", join_keys=["match_id"])
│   │   │   │   ├── 📄 team.py
│   │   │   │   │   └── team = Entity(name="team", join_keys=["team_id"])
│   │   │   │   └── 📄 player.py
│   │   │   │       └── player = Entity(name="player", join_keys=["player_id"])
│   │   │   │
│   │   │   ├── 📁 feature_views/
│   │   │   │   ├── 📄 match_statistics.py
│   │   │   │   │   └── match_statistics = FeatureView(
│   │   │   │   │       name="match_statistics",
│   │   │   │   │       entities=[match],
│   │   │   │   │       schema=[
│   │   │   │   │           Field(name="home_xg", dtype=Float32),
│   │   │   │   │           Field(name="away_xg", dtype=Float32),
│   │   │   │   │           Field(name="home_possession", dtype=Float32),
│   │   │   │   │           Field(name="away_possession", dtype=Float32),
│   │   │   │   │           Field(name="home_shots", dtype=Int32),
│   │   │   │   │           Field(name="away_shots", dtype=Int32),
│   │   │   │   │           Field(name="home_corners", dtype=Int32),
│   │   │   │   │           Field(name="away_corners", dtype=Int32),
│   │   │   │   │       ],
│   │   │   │   │       ttl=timedelta(hours=24)
│   │   │   │   │   )
│   │   │   │   │
│   │   │   │   ├── 📄 live_odds.py
│   │   │   │   │   └── live_odds = FeatureView(
│   │   │   │   │       name="live_odds",
│   │   │   │   │       schema=[
│   │   │   │   │           Field(name="home_win_odds", dtype=Float32),
│   │   │   │   │           Field(name="draw_odds", dtype=Float32),
│   │   │   │   │           Field(name="away_win_odds", dtype=Float32),
│   │   │   │   │           Field(name="over_2_5_odds", dtype=Float32),
│   │   │   │   │           Field(name="btts_yes_odds", dtype=Float32),
│   │   │   │   │           Field(name="implied_home_prob", dtype=Float32),
│   │   │   │   │       ],
│   │   │   │   │       ttl=timedelta(seconds=30)  # Real-time
│   │   │   │   │   )
│   │   │   │   │
│   │   │   │   ├── 📄 team_form.py
│   │   │   │   │   └── team_form = FeatureView(
│   │   │   │   │       name="team_form",
│   │   │   │   │       schema=[
│   │   │   │   │           Field(name="last_5_wins", dtype=Int32),
│   │   │   │   │           Field(name="last_5_goals_scored", dtype=Int32),
│   │   │   │   │           Field(name="last_5_goals_conceded", dtype=Int32),
│   │   │   │   │           Field(name="home_form_score", dtype=Float32),
│   │   │   │   │           Field(name="away_form_score", dtype=Float32),
│   │   │   │   │           Field(name="form_trend", dtype=String),  # UP, DOWN, STABLE
│   │   │   │   │       ],
│   │   │   │   │       ttl=timedelta(days=1)
│   │   │   │   │   )
│   │   │   │   │
│   │   │   │   ├── 📄 head_to_head.py
│   │   │   │   │   └── h2h = FeatureView(
│   │   │   │   │       schema=[
│   │   │   │   │           Field(name="total_meetings", dtype=Int32),
│   │   │   │   │           Field(name="home_wins", dtype=Int32),
│   │   │   │   │           Field(name="away_wins", dtype=Int32),
│   │   │   │   │           Field(name="draws", dtype=Int32),
│   │   │   │   │           Field(name="avg_total_goals", dtype=Float32),
│   │   │   │   │       ]
│   │   │   │   │   )
│   │   │   │   │
│   │   │   │   ├── 📄 graph_embeddings.py
│   │   │   │   │   └── graph_embeddings = FeatureView(
│   │   │   │   │       schema=[
│   │   │   │   │           Field(name="team_embedding", dtype=Array(Float32)),  # 128-dim
│   │   │   │   │           Field(name="match_context_embedding", dtype=Array(Float32)),
│   │   │   │   │       ],
│   │   │   │   │       source=MilvusSource()
│   │   │   │   │   )
│   │   │   │   │
│   │   │   │   └── 📄 confidence_scores.py
│   │   │   │       └── confidence = FeatureView(
│   │   │   │           schema=[
│   │   │   │               Field(name="data_confidence", dtype=Float32),
│   │   │   │               Field(name="freshness_score", dtype=Float32),
│   │   │   │               Field(name="coverage_score", dtype=Float32),
│   │   │   │           ]
│   │   │   │       )
│   │   │   │
│   │   │   └── 📁 on_demand_features/
│   │   │       ├── 📄 vsnr_calculator.py
│   │   │       │   └── @on_demand_feature_view
│   │   │       │       def vsnr_features(inputs):
│   │   │       │           delta_prob = abs(inputs["current_prob"] - inputs["prev_prob"])
│   │   │       │           variance = inputs["prob_variance"]
│   │   │       │           vsnr = delta_prob / sqrt(variance + 1e-6)
│   │   │       │           return {"vsnr": clip(vsnr, 1.5, 3.5)}
│   │   │       │
│   │   │       ├── 📄 cas_calculator.py
│   │   │       │   └── @on_demand_feature_view
│   │   │       │       def cas_features(inputs):
│   │   │       │           vsnr = inputs["vsnr"]
│   │   │       │           decay = 1 / (1 + exp(0.7 * (inputs["minute"] - 85)))
│   │   │       │           cw = inputs["confidence_weight"]
│   │   │       │           cas = (vsnr * decay * cw) / inputs["corridor_liquidity"]
│   │   │       │           return {
│   │   │       │               "cas": cas,
│   │   │       │               "action": "bet" if cas > 1.0 else "hold"
│   │   │       │           }
│   │   │       │
│   │   │       └── 📄 gamma_calculator.py
│   │   │           └── @on_demand_feature_view
│   │   │               def gamma_features(inputs):
│   │   │                   gamma = inputs["market_sensitivity"]
│   │   │                   if gamma < -0.08:
│   │   │                       mode = "COORDINATION"
│   │   │                   elif gamma > 0.52:
│   │   │                       mode = "LEADERSHIP"
│   │   │                   else:
│   │   │                       mode = "NEUTRAL"
│   │   │                   return {"gamma": gamma, "market_mode": mode}
│   │   │
│   │   ├── 📄 pyproject.toml
│   │   └── 📄 README.md
│   │
│   └── 📁 api-gateway/
│       ├── 📁 src/
│       │   ├── 📁 routes/
│       │   │   ├── 📄 matches.py
│       │   │   │   ├── @router.get("/matches")
│       │   │   │   │   └── async def list_matches(league_id, date, status)
│       │   │   │   ├── @router.get("/matches/{match_id}")
│       │   │   │   │   └── async def get_match(match_id) -> MatchDetail
│       │   │   │   ├── @router.get("/matches/{match_id}/statistics")
│       │   │   │   │   └── async def get_statistics(match_id)
│       │   │   │   └── @router.get("/matches/live")
│       │   │   │       └── async def get_live_matches()
│       │   │   │
│       │   │   ├── 📄 predictions.py
│       │   │   │   ├── @router.get("/predictions/{match_id}")
│       │   │   │   │   └── async def get_prediction(match_id) -> Prediction
│       │   │   │   │       ├── response:
│       │   │   │   │       │   ├── home_win_prob: float
│       │   │   │   │       │   ├── draw_prob: float
│       │   │   │   │       │   ├── away_win_prob: float
│       │   │   │   │       │   ├── uncertainty: float (τ)
│       │   │   │   │       │   ├── confidence: float
│       │   │   │   │       │   └── recommendation: str
│       │   │   │   │       └── skip_if: uncertainty > 0.4
│       │   │   │   └── @router.post("/predictions/batch")
│       │   │   │       └── async def batch_predict(match_ids: List[int])
│       │   │   │
│       │   │   ├── 📄 coupons.py
│       │   │   │   ├── @router.post("/coupons/optimize")
│       │   │   │   │   └── async def optimize_coupon(predictions, constraints)
│       │   │   │   │       ├── optimizer: HybridCouponOptimizer
│       │   │   │   │       ├── constraints:
│       │   │   │   │       │   ├── max_selections: 10
│       │   │   │   │       │   ├── min_odds: 1.2
│       │   │   │   │       │   └── risk_budget: float
│       │   │   │   │       └── response: OptimalCoupon
│       │   │   │   ├── @router.get("/coupons/systems")
│       │   │   │   │   └── async def get_system_coupons()
│       │   │   │   │       └── types: [Trixie, Yankee, Heinz, Lucky15...]
│       │   │   │   └── @router.post("/coupons/kelly")
│       │   │   │       └── async def calculate_kelly(coupon) -> KellySizing
│       │   │   │
│       │   │   ├── 📄 strategies.py
│       │   │   │   ├── @router.get("/strategies")
│       │   │   │   │   └── async def list_strategies() -> List[Strategy]
│       │   │   │   ├── @router.get("/strategies/{strategy_id}/performance")
│       │   │   │   │   └── async def get_performance(strategy_id, period)
│       │   │   │   └── @router.post("/strategies/allocate")
│       │   │   │       └── async def allocate_capital(strategies, bankroll)
│       │   │   │
│       │   │   ├── 📄 risk.py
│       │   │   │   ├── @router.get("/risk/metrics")
│       │   │   │   │   └── async def get_risk_metrics() -> RiskDashboard
│       │   │   │   │       ├── var_5: float
│       │   │   │   │       ├── cvar: float
│       │   │   │   │       ├── max_drawdown: float
│       │   │   │   │       ├── sharpe_ratio: float
│       │   │   │   │       └── current_exposure: float
│       │   │   │   ├── @router.get("/risk/limits")
│       │   │   │   │   └── async def get_limits() -> RiskLimits
│       │   │   │   └── @router.get("/risk/circuit-breakers")
│       │   │   │       └── async def get_circuit_breaker_status()
│       │   │   │
│       │   │   ├── 📄 portfolio.py
│       │   │   │   ├── @router.get("/portfolio")
│       │   │   │   │   └── async def get_portfolio() -> Portfolio
│       │   │   │   ├── @router.get("/portfolio/positions")
│       │   │   │   │   └── async def get_open_positions()
│       │   │   │   └── @router.get("/portfolio/pnl")
│       │   │   │       └── async def get_pnl(period: str)
│       │   │   │
│       │   │   └── 📄 health.py
│       │   │       ├── @router.get("/health")
│       │   │       │   └── async def health() -> {"status": "ok"}
│       │   │       ├── @router.get("/ready")
│       │   │       │   └── async def ready() -> check_dependencies()
│       │   │       └── @router.get("/metrics")
│       │   │           └── async def metrics() -> prometheus_format
│       │   │
│       │   ├── 📁 websocket/
│       │   │   ├── 📄 live_updates.py
│       │   │   │   └── class LiveUpdateManager:
│       │   │   │       ├── @websocket("/ws/matches/{match_id}")
│       │   │   │       ├── subscribe(match_id)
│       │   │   │       ├── broadcast_update(match_id, data)
│       │   │   │       └── heartbeat_interval: 30s
│       │   │   │
│       │   │   ├── 📄 prediction_stream.py
│       │   │   │   └── @websocket("/ws/predictions")
│       │   │   │       ├── stream_predictions(filters)
│       │   │   │       └── format: {match_id, probs, uncertainty, timestamp}
│       │   │   │
│       │   │   └── 📄 portfolio_stream.py
│       │   │       └── @websocket("/ws/portfolio")
│       │   │           ├── stream_pnl_updates()
│       │   │           └── stream_position_changes()
│       │   │
│       │   └── 📁 middleware/
│       │       ├── 📄 auth.py
│       │       │   ├── class JWTMiddleware:
│       │       │   │   ├── validate_token(token) -> Claims
│       │       │   │   └── refresh_token(token) -> NewToken
│       │       │   └── class MTLSMiddleware:
│       │       │       └── validate_certificate(cert) -> ServiceIdentity
│       │       │
│       │       ├── 📄 rate_limiter.py
│       │       │   └── class APIRateLimiter:
│       │       │       ├── limits:
│       │       │       │   ├── anonymous: 10 req/min
│       │       │       │   ├── authenticated: 100 req/min
│       │       │       │   └── premium: 1000 req/min
│       │       │       └── backend: redis
│       │       │
│       │       ├── 📄 tracing.py
│       │       │   └── class TracingMiddleware:
│       │       │       ├── provider: OpenTelemetry
│       │       │       ├── exporter: Jaeger
│       │       │       └── propagators: [W3C TraceContext, B3]
│       │       │
│       │       └── 📄 metrics.py
│       │           └── class MetricsMiddleware:
│       │               ├── request_count (counter)
│       │               ├── request_latency (histogram)
│       │               ├── request_size (histogram)
│       │               └── active_connections (gauge)
│       │
│       ├── 📄 pyproject.toml
│       ├── 📄 Dockerfile
│       └── 📄 README.md
```

---

## DEVAMI → PROJECT_TREE_v3.1_PART3.md (ML Katmanı)
