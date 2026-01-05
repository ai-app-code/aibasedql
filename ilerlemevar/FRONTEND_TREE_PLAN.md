# 📋 FRONTEND PROJECT TREE - OLUŞTURMA PLANI

## 🎯 Hedef
FRONTEND_ARCHITECTURE_v2.1.md'deki UI şemasına göre, mevcut PROJECT_TREE_v3.1_PART*.md dosyalarıyla aynı derinlik ve detayda frontend tree yapısı oluşturmak.

---

## 📊 Referans Dosya Analizi

### Mevcut PART Dosyalarının Yapısı:
- **Derinlik:** 4+ seviye (klasör → dosya → class → method → implementation detayları)
- **Format:** ASCII tree karakterleri (├── └── │)
- **Detay Seviyesi:** Her dosyanın içindeki class, method, parametreler, return type'lar
- **Kod Snippet'leri:** Key implementation'lar için kod örnekleri
- **Satır Sayısı:** ~500-600 satır/dosya

### Frontend için Uygulanacak Aynı Prensipler:
1. ✅ Her modülün dosya yapısı gösterilecek
2. ✅ Component'lerin props, state, hooks'ları listelenecek
3. ✅ RxJS stream'lerinin operator chain'leri yazılacak
4. ✅ Go BFF handler'larının request/response tipleri eklenecek
5. ✅ Protobuf message tanımları detaylandırılacak

---

## 🗂️ FRONTEND TREE BÖLÜMLEME PLANI

### BÖLÜM 1: Shell + Shared Infrastructure
**Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_A.md`

```
📁 frontend/
├── 📁 shell/                    ← Webpack Module Federation Host
│   ├── 📄 webpack.config.js
│   │   ├── ModuleFederationPlugin
│   │   │   ├── name: 'shell'
│   │   │   ├── remotes: {signal-center, matches, logs, terminal, ...}
│   │   │   └── shared: {react, rxjs, react-window, xterm}
│   ├── 📄 App.tsx
│   │   ├── RoleProvider
│   │   ├── AuthProvider
│   │   ├── ThemeProvider
│   │   └── RouterConfig
│   ├── 📁 components/
│   │   ├── Header.tsx
│   │   │   ├── RoleTabs
│   │   │   ├── GlobalSearch
│   │   │   ├── SystemStatus
│   │   │   └── UserMenu
│   │   ├── Sidebar.tsx
│   │   │   ├── DynamicMenu (role-based)
│   │   │   └── CollapseToggle
│   │   └── Footer.tsx
│   └── 📁 hooks/
│       ├── useAuth.ts
│       ├── useRole.ts
│       └── usePermissions.ts
│
├── 📁 shared/                   ← Ortak bileşenler (tüm modüller kullanır)
│   ├── 📁 components/
│   │   ├── Card.tsx
│   │   ├── Gauge.tsx
│   │   ├── Donut.tsx
│   │   ├── Sparkline.tsx
│   │   ├── VirtualTable.tsx (react-window)
│   │   ├── Heatmap.tsx (WebGL)
│   │   └── AlertBanner.tsx
│   ├── 📁 hooks/
│   │   ├── useStream.ts         ← RxJS → React bridge
│   │   ├── useSWR.ts            ← Cache hook
│   │   └── usePermission.ts     ← RBAC check
│   ├── 📁 streams/
│   │   ├── index.ts             ← SharedWorker bridge
│   │   ├── alarms.ts
│   │   ├── matches.ts
│   │   ├── broadcast.ts
│   │   ├── agents.ts
│   │   └── circuitBreakers.ts
│   └── 📁 utils/
│       ├── protobuf.ts          ← Decode utilities
│       └── formatting.ts
│
└── 📁 worker/                   ← SharedWorker (cross-tab WS)
    ├── 📄 shared.ts
    │   ├── WebSocket connection manager
    │   ├── BroadcastChannel setup
    │   └── Reconnection logic
    └── 📄 types.ts
```

**Tahmini Satır:** ~300-350

---

### BÖLÜM 2: Operator + Analyst Modülleri
**Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_B.md`

```
📁 frontend/modules/
│
├── 📁 signal-center/            ← Operator default page
│   ├── 📄 RemoteEntry.tsx
│   ├── 📁 components/
│   │   ├── VSNRCASFeed.tsx
│   │   │   └── useStream(alarms$)
│   │   ├── EDLHeatmap.tsx (WebGL)
│   │   ├── BroadcastTicker.tsx
│   │   └── GammaGauge.tsx
│   └── 📁 widgets/
│       └── SignalCenterGrid.tsx
│
├── 📁 matches/
│   ├── 📄 RemoteEntry.tsx
│   ├── 📁 pages/
│   │   ├── MatchList.tsx (virtualized)
│   │   └── MatchDetail.tsx
│   └── 📁 components/
│       ├── Timeline.tsx
│       └── OddsHistory.tsx
│
├── 📁 predictions/
│   ├── 📁 components/
│   │   ├── HRLRecommendations.tsx
│   │   ├── CouponBuilder.tsx
│   │   └── KellyDistribution.tsx
│   └── 📁 hooks/
│       └── usePredictions.ts
│
├── 📁 risk/
│   ├── 📁 components/
│   │   ├── VaRGauge.tsx
│   │   ├── CVaRGauge.tsx
│   │   ├── MaxDrawdown.tsx
│   │   └── CircuitBreakerMatrix.tsx
│   └── 📁 hooks/
│       └── useRiskMetrics.ts
│
├── 📁 strategies/               ← Analyst
│   ├── 📁 components/
│   │   ├── UCBManagerChart.tsx
│   │   ├── StrategyWeightSliders.tsx
│   │   └── BCDRegimeChart.tsx
│   └── 📁 hooks/
│       └── useStrategies.ts
│
├── 📁 backtest/                 ← Analyst (YENİ v2.1)
│   ├── 📁 components/
│   │   ├── PerformanceChart.tsx
│   │   ├── StrategyComparison.tsx
│   │   └── EquityCurve.tsx
│   └── 📁 hooks/
│       └── useBacktest.ts
│
└── 📁 reports/                  ← Analyst (YENİ v2.1)
    ├── 📁 components/
    │   ├── DailySummary.tsx
    │   ├── WeeklyReport.tsx
    │   └── ExportButton.tsx
    └── 📁 utils/
        └── exportToPDF.ts
```

**Tahmini Satır:** ~350-400

---

### BÖLÜM 3: DevOps Modülleri (Logs ⭐)
**Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_C.md`

```
📁 frontend/modules/
│
├── 📁 metrics/                  ← DevOps
│   ├── 📁 components/
│   │   ├── PrometheusPanel.tsx
│   │   └── BusinessKPIs.tsx
│   └── 📁 hooks/
│       └── useMetrics.ts
│
├── 📁 logs/                     ← DevOps (KRİTİK YENİ v2.1) ⭐⭐⭐
│   ├── 📄 RemoteEntry.tsx
│   ├── 📁 components/
│   │   ├── LogStream.tsx
│   │   │   ├── import { FixedSizeList } from 'react-window'
│   │   │   ├── useLogs() hook
│   │   │   └── Virtualized rendering (10k cap)
│   │   ├── SearchPanel.tsx
│   │   │   ├── ServiceDropdown
│   │   │   ├── LevelChips (DEBUG/INFO/WARN/ERROR/FATAL)
│   │   │   └── TimeRangePicker
│   │   └── StatsBar.tsx
│   │       └── logs/s, total, lag, buffer
│   ├── 📁 hooks/
│   │   ├── useLogs.ts
│   │   │   └── useSyncExternalStore(logs$)
│   │   └── useLogBackfill.ts
│   │       └── GET /api/logs/backfill
│   ├── 📁 streams/
│   │   └── logs.ts
│   │       ├── EWMA Rate Calculator
│   │       ├── Hybrid pattern (rate < 1000 vs ≥1000)
│   │       ├── auditTime(100) + windowCount(500)
│   │       └── observeOn(animationFrameScheduler)
│   └── 📄 styles.css
│       └── Log level styling (debug/info/warn/error/fatal)
│
├── 📁 traces/
│   └── 📁 components/
│       ├── JaegerEmbed.tsx
│       └── SpanVisualization.tsx
│
├── 📁 alerts/                   ← DevOps (YENİ v2.1)
│   └── 📁 components/
│       ├── ActiveAlerts.tsx
│       ├── AlertHistory.tsx
│       └── AcknowledgeButton.tsx
│
└── 📁 health/
    └── 📁 components/
        ├── ServiceStatus.tsx
        └── DependencyGraph.tsx (D3.js)
```

**Tahmini Satır:** ~350-400

---

### BÖLÜM 4: Infrastructure Modülleri (Terminal ⭐)
**Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_D.md`

```
📁 frontend/modules/
│
├── 📁 k8s/                      ← Infrastructure
│   ├── 📁 components/
│   │   ├── ClusterOverview.tsx
│   │   ├── PodsTable.tsx
│   │   ├── DeploymentsTable.tsx
│   │   └── HPAStatus.tsx
│   └── 📁 hooks/
│       └── useK8s.ts
│
├── 📁 argocd/
│   └── 📁 components/
│       ├── ApplicationsList.tsx
│       ├── SyncStatus.tsx
│       ├── CanaryProgress.tsx
│       └── RollbackButton.tsx
│
├── 📁 terminal/                 ← Infrastructure (KRİTİK YENİ v2.1) ⭐⭐⭐
│   ├── 📄 RemoteEntry.tsx
│   ├── 📁 components/
│   │   ├── Terminal.tsx
│   │   │   ├── import { Terminal } from 'xterm'
│   │   │   ├── import { FitAddon } from 'xterm-addon-fit'
│   │   │   ├── useTerminal() hook
│   │   │   └── WebSocket connection
│   │   ├── PodSelector.tsx
│   │   │   └── GET /api/k8s/pods
│   │   ├── TerminalTabs.tsx
│   │   │   └── Multiple session support
│   │   ├── ReadOnlyToggle.tsx
│   │   │   └── Interactive ⇄ Read-Only
│   │   └── SessionInfo.tsx
│   │       └── Session ID, Connected time, Idle timeout
│   ├── 📁 hooks/
│   │   ├── useTerminal.ts
│   │   │   ├── WebSocket setup (wss://bff/ws/terminal)
│   │   │   ├── xterm.js initialization
│   │   │   ├── Resize handling
│   │   │   └── Auto-reconnect
│   │   └── useCommandHistory.ts
│   ├── 📁 utils/
│   │   └── audit.ts
│   │       └── Send to audit.terminal.commands topic
│   └── 📄 styles.css
│       └── xterm.js theming (Deep Space)
│
└── 📁 config/
    └── 📁 components/
        ├── ConsulBrowser.tsx
        ├── ConfigDiff.tsx (Monaco Editor)
        └── VaultSecrets.tsx (masked)
```

**Tahmini Satır:** ~350-400

---

### BÖLÜM 5: MLOps + Admin + BFF
**Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_E.md`

```
📁 frontend/modules/
│
├── 📁 experiments/              ← MLOps
│   └── 📁 components/
│       ├── MLflowEmbed.tsx
│       │   ├── iframe + SSO token injection
│       │   ├── Token refresh T-45s
│       │   └── postMessage bridge
│       └── ActionButtons.tsx
│
├── 📁 training/
│   └── 📁 components/
│       ├── RayJobList.tsx
│       └── GPUUsageGauges.tsx
│
├── 📁 features/
│   └── 📁 components/
│       ├── FeatureStoreBrowser.tsx
│       └── FeatureLineage.tsx (DAG)
│
├── 📁 quality/
│   └── 📁 components/
│       ├── ValidationRunHistory.tsx
│       └── DataProfiles.tsx
│
├── 📁 hyperparams/              ← MLOps (YENİ v2.1) ⭐
│   └── 📁 components/
│       ├── StudyList.tsx
│       ├── TrialHistory.tsx
│       ├── BestParameters.tsx
│       └── OptimizationChart.tsx
│
├── 📁 users/                    ← Admin
│   └── 📁 components/
│       ├── UserList.tsx
│       ├── RoleAssignment.tsx
│       └── PermissionMatrix.tsx
│
├── 📁 flags/
│   └── 📁 components/
│       ├── FlagList.tsx
│       └── ToggleSwitch.tsx
│
├── 📁 audit/                    ← Admin (YENİ v2.1) ⭐
│   ├── 📁 components/
│   │   ├── AuditTimeline.tsx
│   │   │   ├── Keyset pagination (ts, id)
│   │   │   ├── "Load More" button
│   │   │   └── Virtualized (react-window)
│   │   ├── FilterPanel.tsx
│   │   └── ExportButton.tsx
│   └── 📁 hooks/
│       └── useAuditLogs.ts
│           └── GET /api/audit
│
├── 📁 secrets/
│   └── 📁 components/
│       ├── VaultBrowser.tsx
│       └── MaskedValue.tsx
│
└── 📁 system-config/
    └── 📁 components/
        ├── GlobalSettings.tsx
        ├── APIKeysManagement.tsx
        └── BroadcastConfig.tsx
│
├── 📁 proto/                    ← Protobuf Schemas
│   ├── 📄 superbet.proto
│   │   ├── TwinDelta
│   │   ├── AlarmEvent
│   │   ├── MatchUpdate
│   │   ├── BroadcastEvent
│   │   ├── AgentStatus
│   │   ├── CircuitBreakerStatus
│   │   ├── LogEntry ⭐
│   │   └── TerminalData ⭐
│   └── 📄 compile.sh
│       └── protoc --js_out=...
│
└── 📁 bff/                      ← Go BFF (Backend for Frontend)
    ├── 📄 main.go
    │   ├── mTLS setup
    │   ├── Vault integration
    │   └── HTTP/WS handlers
    ├── 📁 handlers/
    │   ├── ws_alarms.go
    │   ├── ws_matches.go
    │   ├── ws_broadcast.go
    │   ├── ws_logs.go ⭐
    │   │   └── Loki stream → WebSocket
    │   ├── ws_terminal.go ⭐⭐⭐
    │   │   ├── kubectl exec → xterm.js
    │   │   ├── Audit to Kafka
    │   │   └── 15min idle timeout
    │   └── rest_mlflow.go
    │       └── GET /api/mlflow/token
    ├── 📁 kafka/
    │   ├── consumer.go
    │   └── producer.go
    │       └── audit.terminal.commands
    └── 📁 auth/
        ├── mtls.go
        └── jwt.go
```

**Tahmini Satır:** ~400-450

---

## 📊 TOPLAM PLANLAMA

| Bölüm | Dosya Adı | İçerik | Tahmini Satır |
|-------|-----------|--------|---------------|
| **A** | `PROJECT_TREE_v3.1_PART6_Frontend_A.md` | Shell + Shared Infrastructure | ~350 |
| **B** | `PROJECT_TREE_v3.1_PART6_Frontend_B.md` | Operator + Analyst | ~400 |
| **C** | `PROJECT_TREE_v3.1_PART6_Frontend_C.md` | DevOps (Logs ⭐) | ~400 |
| **D** | `PROJECT_TREE_v3.1_PART6_Frontend_D.md` | Infrastructure (Terminal ⭐) | ~400 |
| **E** | `PROJECT_TREE_v3.1_PART6_Frontend_E.md` | MLOps + Admin + BFF | ~450 |

**Toplam:** ~2000 satır, 5 dosya

---

## ✅ ONAY SONRASI İŞLEM ADIMLARI

1. ✅ Plan onaylandı mı? → Kullanıcı "evet" derse devam
2. 📝 PART6_Frontend_A.md oluştur (Shell + Shared)
3. 📝 PART6_Frontend_B.md oluştur (Operator + Analyst)
4. 📝 PART6_Frontend_C.md oluştur (DevOps + Logs)
5. 📝 PART6_Frontend_D.md oluştur (Infrastructure + Terminal)
6. 📝 PART6_Frontend_E.md oluştur (MLOps + Admin + BFF)
7. 🔍 Her dosyayı kullanıcı ile birlikte check et
8. ✅ Tamamlandı!

---

**Hazırlayan:** Antigravity Agent  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md + PROJECT_TREE_v3.1_PART*.md pattern
