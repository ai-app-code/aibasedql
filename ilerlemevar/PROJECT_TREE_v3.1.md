# 🏗️ SUPERBET GENESIS v3.1 - DERİN PROJE TREE YAPISI
## Enterprise-Grade AI Betting System - 4+ Seviye Derinlik

**Tarih:** 03.01.2026 | **Referans:** bettingenesis-v3.1.md

---

```
superbet-genesis/
│
├── 📁 .github/
│   ├── 📁 workflows/
│   │   ├── 📄 ci.yml
│   │   │   ├── jobs:
│   │   │   │   ├── lint: [ruff, mypy, black]
│   │   │   │   ├── test: [pytest, coverage >80%]
│   │   │   │   ├── security: [bandit, safety, trivy]
│   │   │   │   └── build: [docker, push registry]
│   │   │   └── triggers: [push, PR]
│   │   │
│   │   ├── 📄 cd.yml
│   │   │   ├── jobs:
│   │   │   │   ├── deploy-staging: [helm upgrade]
│   │   │   │   ├── integration-test: [k6, pytest-integration]
│   │   │   │   ├── canary-deploy: [argo rollouts, %10 traffic]
│   │   │   │   └── promote-prod: [manual approval, %100]
│   │   │   └── rollback:
│   │   │       ├── trigger: [p99 > 60ms, error_rate > 1%]
│   │   │       └── action: [argo rollout undo, <1dk]
│   │   │
│   │   ├── 📄 model-validation.yml
│   │   │   ├── jobs:
│   │   │   │   ├── calibration-check:
│   │   │   │   │   ├── pit_uniformity: [KS test, p>0.05]
│   │   │   │   │   ├── ece: [<0.05]
│   │   │   │   │   └── reliability_slope: [≈1]
│   │   │   │   ├── latency-benchmark: [p99 <40ms]
│   │   │   │   └── drift-check: [evidently report]
│   │   │   └── gates:
│   │   │       ├── pass: [deploy allowed]
│   │   │       └── fail: [block, alert]
│   │   │
│   │   └── 📄 security-scan.yml
│   │       ├── sast: [semgrep, codeql]
│   │       ├── dast: [owasp zap]
│   │       ├── container: [trivy, grype]
│   │       └── secrets: [gitleaks, trufflehog]
│   │
│   └── 📁 ISSUE_TEMPLATE/
│       ├── 📄 bug_report.md
│       ├── 📄 feature_request.md
│       ├── 📄 model_drift_alert.md
│       └── 📄 circuit_breaker_incident.md
│
├── 📁 infrastructure/
│   ├── 📁 terraform/
│   │   ├── 📁 modules/
│   │   │   ├── 📁 kubernetes/
│   │   │   │   ├── 📄 cluster.tf
│   │   │   │   │   ├── node_pools:
│   │   │   │   │   │   ├── general: [4 vCPU, 16GB, 3 nodes]
│   │   │   │   │   │   ├── ml-inference: [8 vCPU, 32GB, GPU, 2 nodes]
│   │   │   │   │   │   └── data-processing: [8 vCPU, 64GB, 3 nodes]
│   │   │   │   │   └── addons: [istio, prometheus, argocd]
│   │   │   │   ├── 📄 networking.tf
│   │   │   │   │   ├── vpc: [10.0.0.0/16]
│   │   │   │   │   ├── subnets: [public, private, data]
│   │   │   │   │   └── security_groups: [ingress, egress rules]
│   │   │   │   └── 📄 iam.tf
│   │   │   │       ├── roles: [cluster-admin, ml-operator, data-engineer]
│   │   │   │       └── service_accounts: [kserve, flink, feast]
│   │   │   │
│   │   │   ├── 📁 databases/
│   │   │   │   ├── 📄 clickhouse.tf
│   │   │   │   │   ├── cluster:
│   │   │   │   │   │   ├── shards: 3
│   │   │   │   │   │   ├── replicas: 2
│   │   │   │   │   │   └── storage: 1TB NVMe
│   │   │   │   │   ├── tables:
│   │   │   │   │   │   ├── matches: [ReplacingMergeTree]
│   │   │   │   │   │   ├── odds: [ReplacingMergeTree]
│   │   │   │   │   │   ├── predictions: [MergeTree, TTL 90d]
│   │   │   │   │   │   └── events: [Kafka Engine]
│   │   │   │   │   └── materialized_views:
│   │   │   │   │       ├── mv_match_stats
│   │   │   │   │       ├── mv_team_form
│   │   │   │   │       └── mv_hourly_aggregates
│   │   │   │   │
│   │   │   │   ├── 📄 timescaledb.tf
│   │   │   │   │   ├── hypertables:
│   │   │   │   │   │   ├── live_events: [chunk_time: 1h]
│   │   │   │   │   │   ├── odds_history: [chunk_time: 1d]
│   │   │   │   │   │   └── predictions: [chunk_time: 1d]
│   │   │   │   │   └── continuous_aggregates:
│   │   │   │   │       ├── cagg_5min_stats
│   │   │   │   │       └── cagg_hourly_summary
│   │   │   │   │
│   │   │   │   ├── 📄 redis.tf
│   │   │   │   │   ├── cluster_mode: enabled
│   │   │   │   │   ├── shards: 6
│   │   │   │   │   ├── replicas_per_shard: 1
│   │   │   │   │   └── data_structures:
│   │   │   │   │       ├── cache: [match_data, TTL 30s]
│   │   │   │   │       ├── rate_limiter: [token_bucket, Lua scripts]
│   │   │   │   │       ├── feature_store: [online features]
│   │   │   │   │       └── state_store: [agent states, WATCH-MULTI]
│   │   │   │   │
│   │   │   │   ├── 📄 neo4j.tf
│   │   │   │   │   ├── causal_cluster: 3 nodes
│   │   │   │   │   └── indexes:
│   │   │   │   │       ├── Team(id)
│   │   │   │   │       ├── Player(id)
│   │   │   │   │       └── Match(id, date)
│   │   │   │   │
│   │   │   │   └── 📄 milvus.tf
│   │   │   │       ├── cluster_mode: distributed
│   │   │   │       └── collections:
│   │   │   │           ├── team_embeddings: [dim=128, HNSW]
│   │   │   │           ├── match_embeddings: [dim=128, IVF_FLAT]
│   │   │   │           └── strategy_embeddings: [dim=64, HNSW]
│   │   │   │
│   │   │   ├── 📁 kafka/
│   │   │   │   ├── 📄 cluster.tf
│   │   │   │   │   ├── brokers: 3
│   │   │   │   │   ├── replication_factor: 3
│   │   │   │   │   └── min_insync_replicas: 2
│   │   │   │   └── 📄 topics.tf
│   │   │   │       ├── football.match.update: [partitions: 12, retention: 7d]
│   │   │   │       ├── football.odds.update: [partitions: 24, retention: 3d]
│   │   │   │       ├── football.live.events: [partitions: 48, retention: 1d]
│   │   │   │       ├── predictions.output: [partitions: 12, retention: 30d]
│   │   │   │       ├── graph.events: [partitions: 6, retention: 7d]
│   │   │   │       ├── sentiment.events: [partitions: 6, retention: 3d]
│   │   │   │       └── system.checkpoints: [partitions: 3, retention: 90d, compacted]
│   │   │   │
│   │   │   ├── 📁 monitoring/
│   │   │   │   ├── 📄 prometheus.tf
│   │   │   │   │   ├── retention: 30d
│   │   │   │   │   ├── storage: 500GB
│   │   │   │   │   └── scrape_configs:
│   │   │   │   │       ├── kubernetes-pods
│   │   │   │   │       ├── kserve-metrics
│   │   │   │   │       └── custom-exporters
│   │   │   │   ├── 📄 grafana.tf
│   │   │   │   │   ├── datasources: [prometheus, clickhouse, jaeger]
│   │   │   │   │   └── dashboards: [auto-provisioned]
│   │   │   │   ├── 📄 jaeger.tf
│   │   │   │   │   ├── collector: 3 replicas
│   │   │   │   │   ├── storage: elasticsearch
│   │   │   │   │   └── sampling_rate: 0.1
│   │   │   │   └── 📄 alertmanager.tf
│   │   │   │       ├── receivers: [slack, pagerduty, email]
│   │   │   │       └── routes:
│   │   │   │           ├── critical: [pagerduty, 5min]
│   │   │   │           ├── warning: [slack, 15min]
│   │   │   │           └── info: [email, 1h]
│   │   │   │
│   │   │   ├── 📁 security/
│   │   │   │   ├── 📄 vault.tf
│   │   │   │   │   ├── seal: awskms
│   │   │   │   │   ├── auth_methods: [kubernetes, approle]
│   │   │   │   │   └── secrets_engines:
│   │   │   │   │       ├── kv-v2: [api_keys, db_credentials]
│   │   │   │   │       ├── pki: [internal CAs]
│   │   │   │   │       └── transit: [encryption keys]
│   │   │   │   ├── 📄 istio.tf
│   │   │   │   │   ├── mtls: STRICT
│   │   │   │   │   ├── peer_authentication: namespace-wide
│   │   │   │   │   └── authorization_policies:
│   │   │   │   │       ├── deny-all-default
│   │   │   │   │       └── allow-specific-services
│   │   │   │   └── 📄 spire.tf
│   │   │   │       ├── server: [trust_domain: superbet.local]
│   │   │   │       └── agents: [daemonset on all nodes]
│   │   │   │
│   │   │   └── 📁 ml-platform/
│   │   │       ├── 📄 kserve.tf
│   │   │       │   ├── inference_services:
│   │   │       │   │   ├── layer1-lightgbm: [cpu, 2 replicas]
│   │   │       │   │   ├── layer2-hypernetwork: [gpu, 2 replicas]
│   │   │       │   │   ├── layer3-edl: [gpu, 3 replicas, canary]
│   │   │       │   │   └── ensemble-prematch: [gpu, 2 replicas]
│   │   │       │   └── transformers: [pre/post processing]
│   │   │       ├── 📄 triton.tf
│   │   │       │   ├── model_repository: s3://models/
│   │   │       │   ├── optimization: FP16
│   │   │       │   └── batching: dynamic, max_batch_size=32
│   │   │       ├── 📄 mlflow.tf
│   │   │       │   ├── tracking_server: [postgresql backend]
│   │   │       │   ├── artifact_store: s3://mlflow-artifacts/
│   │   │       │   └── model_registry: enabled
│   │   │       └── 📄 ray.tf
│   │   │           ├── head_node: [8 vCPU, 32GB]
│   │   │           ├── worker_nodes: [autoscale 2-10]
│   │   │           └── dashboard: enabled
│   │   │
│   │   └── 📁 environments/
│   │       ├── 📄 dev.tfvars
│   │       ├── 📄 staging.tfvars
│   │       └── 📄 prod.tfvars
│   │
│   └── 📁 helm/
│       └── 📁 charts/
│           └── 📁 superbet-core/
│               ├── 📄 values.yaml
│               │   ├── global:
│               │   │   ├── image.registry: ghcr.io/superbet
│               │   │   ├── env: production
│               │   │   └── replicas: 3
│               │   ├── services:
│               │   │   ├── data-plant: {cpu: 2, memory: 4Gi}
│               │   │   ├── api-gateway: {cpu: 1, memory: 2Gi}
│               │   │   └── stream-processor: {cpu: 4, memory: 8Gi}
│               │   └── monitoring:
│               │       ├── prometheus.enabled: true
│               │       └── jaeger.enabled: true
│               └── 📁 templates/
│                   ├── 📄 deployment.yaml
│                   ├── 📄 service.yaml
│                   ├── 📄 hpa.yaml
│                   │   ├── minReplicas: 2
│                   │   ├── maxReplicas: 10
│                   │   └── metrics: [cpu: 70%, memory: 80%, custom: p99_latency]
│                   ├── 📄 pdb.yaml
│                   │   └── minAvailable: 1
│                   └── 📄 servicemonitor.yaml
```

---

## DEVAMI İKİNCİ DOSYADA → PROJECT_TREE_v3.1_PART2.md
