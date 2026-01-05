# 🎨 SUPERBET GENESIS v3.1 - FRONTEND MİMARİ PLANI
## Süper-Rasyonel Dijital Bahis Varlığı - Enterprise UI Blueprint

**Oluşturma:** 04.01.2026  
**Güncelleme:** 04.01.2026 (Full-Stack UI Münazarası Sonucu)  
**Kaynak:** TETRA AI Panel Frontend Münazarası - 2 Oturum (3 Tur + 5 Tur), 10 LLM  
**Versiyon:** v2.1 (Rol Bazlı Command Center + Logs + Terminal + ML Platform + RBAC)  
**Panel Katılımcıları:** Nexus (Mod), Gemini 2.5/3 Flash/Pro, GPT-5, GPT-4o-mini, Grok 4.1, Kimi K2, Qwen3, GLM 4.7, MiniMax M2.1

> ⚡ **KONSEPT:** Role-Based Command Center. Signal-First Operator UI + Infrastructure/MLPlatform controls. Go BFF with mTLS, RxJS stream tabanlı real-time, xterm.js terminal/SSH integration, RBAC enforcement.

> ⚠️ **v2.1 CHANGELOG:** 
> - v2.0 → v2.1: 6 Rol bazlı multi-dashboard (Operator/Analyst/DevOps/Infra/MLOps/Admin)
> - **Logs UI** eklendi: Loki stream + virtualized list + auditTime(100) pattern
> - **Terminal/SSH** eklendi: xterm.js + WebSocket + audit.terminal.commands Kafka topic
> - **ML Platform** eklendi: iframe + SSO Phase-1, MLflow/Ray/Feast/GE entegrasyonu
> - **RBAC Matrisi** eklendi: 6 rol için permission kontrolü
> - **Audit Log Retention** belirlendi: UI 90d (ClickHouse), Archive 180d (S3)

---

# 📌 BÖLÜM 0: VİZYON VE FELSEFE

## Kritik Tasarım Kararları

| Soru | v2.0 Karar | v2.1 Karar | Gerekçe |
|------|------------|------------|---------|
| **Mimari?** | Modüler SPA | **Rol Bazlı Modüler SPA** | 6 farklı rol için optimize edilmiş deneyim |
| **BFF var mı?** | Go BFF zorunlu | **Go BFF zorunlu** | mTLS/Vault, Kafka proxy, Terminal proxy |
| **State?** | RxJS Streams | **RxJS Streams** | 1M/s Kafka akışı + Log streaming |
| **Veri formatı?** | Protobuf (Binary) | **Protobuf (Binary)** | %60 payload azalma |
| **UX felsefesi?** | Signal-first | **Role-first + Signal-first** | Her rol kendi dashboard'unu görür |
| **Render?** | DOM + WebGL | **DOM + WebGL + xterm.js** | Terminal için xterm.js eklendi |
| **Worker?** | SharedWorker | **SharedWorker** | Sekmeler arası tek WS |
| **Log Pattern?** | N/A | **auditTime(100) + windowCount(500)** | Burst korumalı, son log garantili |
| **ML Embed?** | N/A | **iframe + SSO Phase-1** | MVP hızı, token refresh T-45s |
| **Terminal?** | N/A | **xterm.js + WebSocket** | 15dk timeout, audit logging |

## SLO Hedefleri

| Metrik | Hedef | Alert Threshold |
|--------|-------|-----------------|
| **TTI (Time to Interactive)** | < 3s | > 5s |
| **p99 WS Latency** | < 60ms | > 80ms |
| **FPS** | > 60fps | < 50fps |
| **JS Bundle (gzip)** | < 250KB | > 400KB |
| **Log Stream Latency** | < 100ms | > 200ms |
| **Terminal Response** | < 50ms | > 100ms |
| **WCAG Uyumluluk** | 2.1 AA | - |

---

# 🏗️ BÖLÜM 1: ROL BAZLI SAYFA MİMARİSİ

## Genel Yapı

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SUPERBET GENESIS v3.1 - COMMAND CENTER                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  [Operator] [Analyst] [DevOps] [Infra] [MLOps] [Admin]  ← Rol Sekmeleri │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────┐  ┌────────────────────────────────────────────────┐  │
│  │   SIDEBAR    │  │              MIDDLE AREA                       │  │
│  │  (Dynamic)   │  │         (Rol-Spesifik İçerik)                  │  │
│  │              │  │                                                 │  │
│  │  Seçili role │  │  Widget Grid                                   │  │
│  │  göre menü   │  │  RxJS Streams                                  │  │
│  │  değişir     │  │  Real-time Data                                │  │
│  └──────────────┘  └────────────────────────────────────────────────┘  │
│                                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  FOOTER: [Build: abc123] [Model: v3.1] [Canary: 10%] [Env: prod]       │
└─────────────────────────────────────────────────────────────────────────┘
```

## 6 Rol ve Modülleri

### 1. 📊 OPERATOR DASHBOARD
**Odak:** Signal-First Command Center - Hız ve Karar Destek

```
├── Signal Center (Anasayfa - Default)
│   ├── VSNR/CAS Alarm Feed (Real-time RxJS Stream)
│   ├── EDL Uncertainty Heatmap (WebGL Canvas)
│   ├── γ Gamma Rejim Göstergesi (Liderlik/Eşgüdüm)
│   └── Broadcast Live Ticker (Twitter/Telegram Status)
├── Matches
│   ├── Pre-match List (Virtualized Table)
│   ├── Live Matches (Real-time Updates)
│   ├── Match Detail (Timeline, xG, Odds History)
│   └── Pre→Live Handover Status
├── Predictions
│   ├── HRL Önerileri (Top Picks)
│   ├── Kupon Kombinasyonları (Trixie, Yankee, etc.)
│   ├── Kelly Pay Dağılımı (Fractional 0.75)
│   └── Broadcast Yayın Durumu
└── Risk
    ├── VaR/CVaR/MaxDD/Sharpe Gauges
    ├── Risk Limitleri (Günlük 5% / Haftalık 10%)
    └── Circuit Breaker Status Matrix
```

**Routes:** 
- `/operator/signal` (Default landing page)
- `/operator/matches`
- `/operator/matches/:id`
- `/operator/predictions`
- `/operator/risk`

**Widget Grid (Signal Center):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| VSNR/CAS Alarm Feed | x:0 y:0 | w:8 h:12 | WS `/ws/alarms` | Virtualized List |
| Live Matches Ticker | x:8 y:0 | w:4 h:12 | WS `/ws/matches` | Cards |
| Risk Limit Bars | x:8 y:12 | w:4 h:6 | WS `/ws/risk` | Progress Bars |
| Broadcast Live Ticker | x:0 y:24 | w:12 h:2 | WS `/ws/broadcast` | Horizontal Scroll |
| Gamma Regime | x:0 y:12 | w:4 h:6 | WS `/ws/agents` | Gauge |
| CB Status Donut | x:4 y:12 | w:4 h:6 | WS `/ws/circuit-breakers` | Donut Chart |

---

### 2. 📈 ANALYST DASHBOARD
**Odak:** Derin Analiz ve Strateji Yönetimi

```
├── Strategies
│   ├── UCB Manager ROI Grafiği (Time Series)
│   ├── Strateji Ağırlıkları (10+ strateji slider)
│   ├── BCD Rejim Algılama (p_BCD trend line)
│   └── Mode Matrix (Liderlik/Eşgüdüm/Nötr)
├── Backtest (YENİ v2.1)
│   ├── Tarihsel Performans Analizi
│   ├── Strateji Karşılaştırma (Side-by-side)
│   ├── Drawdown Analizi
│   └── Equity Curve
└── Reports (YENİ v2.1)
    ├── Günlük Özet
    ├── Haftalık Rapor
    └── Export (PDF/CSV)
```

**Routes:**
- `/analyst/strategies`
- `/analyst/backtest`
- `/analyst/reports`

**Widget Grid (Strategies):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| ROI/Sharpe Chart | x:0 y:0 | w:12 h:10 | REST `/api/strategies` | Line Chart |
| Strategy Weights | x:0 y:10 | w:8 h:6 | REST | Slider Grid |
| Mode Matrix | x:8 y:10 | w:4 h:6 | WS `/ws/agents` | Heatmap |

---

### 3. 🔧 DEVOPS DASHBOARD
**Odak:** Observability ve Logs

```
├── Metrics
│   ├── Prometheus Dashboard (Mirror Panels)
│   ├── Business KPI'lar (ROI, Win Rate Gauges)
│   └── SLO Dashboard (p99 latency, uptime charts)
├── Logs (YENİ v2.1 - KRİTİK) ⭐
│   ├── Real-time Log Stream (tail -f benzeri)
│   ├── Search/Filter (Lucene query, service, level, time range)
│   ├── Log Levels (DEBUG/INFO/WARN/ERROR/FATAL chips)
│   └── Log Aggregation (Loki backend)
├── Traces
│   ├── Jaeger UI (Embed veya deep-link)
│   ├── Span Visualization (Tree view)
│   └── Latency Distribution (Histogram)
├── Alerts (YENİ v2.1)
│   ├── Active Alerts List
│   ├── Alert History
│   ├── Acknowledge/Silence Actions
│   └── PagerDuty/Slack Integration Status
└── Health
    ├── Service Status (UP/DOWN/DEGRADED cards)
    ├── Uptime Statistics
    └── Dependency Graph (D3.js)
```

**Routes:**
- `/devops/metrics`
- `/devops/logs`
- `/devops/traces`
- `/devops/alerts`
- `/devops/health`

**Widget Grid (Logs - Split View):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| Log Stream | x:0 y:0 | w:8 h:20 (70%) | WS `/ws/logs` | Virtualized List |
| Search Panel | x:8 y:0 | w:4 h:20 (30%) | REST | Form + Chips |
| Stats Bar | x:0 y:20 | w:12 h:4 | WS | Counters |

---

### 4. 🏗️ INFRASTRUCTURE DASHBOARD (YENİ v2.1) ⭐
**Odak:** K8s ve Terminal Yönetimi

```
├── Kubernetes
│   ├── Cluster Overview (SVG Map)
│   ├── Pods Status (Running/Pending/Failed Table)
│   ├── Deployments List
│   ├── HPA Status (current/min/max replicas bars)
│   ├── Resource Usage (CPU/Memory per pod sparklines)
│   └── Restart Counts
├── ArgoCD
│   ├── Applications List
│   ├── Sync Status (Synced/OutOfSync/Unknown badges)
│   ├── Canary Deployment Progress (%10 → %100 bar)
│   ├── Rollback Button (one-click with confirmation)
│   └── Git Commit History
├── Terminal/SSH (YENİ v2.1 - KRİTİK) ⭐⭐⭐
│   ├── Pod Selection Dropdown
│   ├── Interactive Shell (xterm.js + WebSocket)
│   ├── Command History (Session-based)
│   ├── Multiple Tab Support
│   └── Log Streaming (kubectl logs -f)
└── Config
    ├── Consul/Etcd Key-Value Browser (Tree View)
    ├── Config Diff View (Monaco Editor)
    ├── Edit & Apply (with audit confirmation)
    └── Vault Secrets (Masked View, copy-to-clipboard)
```

**Routes:**
- `/infra/k8s`
- `/infra/argocd`
- `/infra/terminal`
- `/infra/config`

**Widget Grid (Terminal):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| Pod Selector | x:0 y:0 | w:12 h:2 | REST `/api/k8s/pods` | Dropdown |
| Terminal Tabs | x:0 y:2 | w:12 h:22 | WS `/ws/terminal` | xterm.js |

---

### 5. 🤖 ML PLATFORM DASHBOARD (YENİ v2.1) ⭐
**Odak:** ML Lifecycle Yönetimi

```
├── Experiments (MLflow)
│   ├── Experiment List
│   ├── Run Comparison (Metrics side-by-side)
│   ├── Model Registry (Versions, Stages)
│   ├── Artifacts Browser
│   └── Promote to Production Button (Admin onayı)
├── Training (Ray.io)
│   ├── Active Jobs List
│   ├── Job Details (Progress bar, logs)
│   ├── GPU/CPU Usage per Job (Gauges)
│   ├── Queue Status
│   └── Cancel/Restart Actions
├── Features (Feast)
│   ├── Feature Store Browser (Tree)
│   ├── Feature Definitions
│   ├── Freshness Status (TTL, last update timestamps)
│   ├── Usage Statistics
│   └── Feature Lineage (DAG visualization)
├── Data Quality (Great Expectations)
│   ├── Validation Run History
│   ├── Failed Checks List
│   ├── Data Profiles
│   ├── Expectation Suite Editor
│   └── Alert on Failure
└── Hyperparameters (Optuna) ⭐
    ├── Study List
    ├── Trial History (Table + Chart)
    ├── Best Parameters View
    ├── Optimization Progress Chart
    └── Parameter Importance (Bar Chart)
```

**Routes:**
- `/mlops/experiments`
- `/mlops/training`
- `/mlops/features`
- `/mlops/quality`
- `/mlops/hyperparams` ⭐

**Widget Grid (Experiments - iframe):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| MLflow iframe | x:0 y:0 | w:12 h:20 | SSO Token | iframe |
| Action Buttons | x:0 y:20 | w:12 h:4 | REST | Buttons |

---

### 6. ⚙️ ADMIN DASHBOARD
**Odak:** Güvenlik, RBAC ve Konfigürasyon

```
├── Users (RBAC)
│   ├── User List (Table with roles)
│   ├── Role Assignment (Operator/Analyst/DevOps/Infra/MLOps/Admin)
│   ├── Permission Matrix (Grid view)
│   ├── Invite/Deactivate User
│   └── Session Management (Kill switch)
├── Feature Flags (LaunchDarkly)
│   ├── Flag List
│   ├── Enable/Disable Toggle
│   ├── Targeting Rules
│   └── Rollout Percentage
├── Audit Logs (YENİ v2.1) ⭐
│   ├── Who did What When (Timeline)
│   ├── Filter by User/Action/Resource
│   ├── Export Audit Trail
│   └── Compliance Reports
├── Secrets (Vault)
│   ├── Secret Paths Browser (Tree)
│   ├── Masked Value View
│   ├── Rotation Status
│   └── Access History
└── System Config
    ├── Global Settings
    ├── API Keys Management
    ├── Rate Limit Config
    └── Broadcast Platform Config (Twitter/Telegram/Android)
```

**Routes:**
- `/admin/users`
- `/admin/flags`
- `/admin/audit`
- `/admin/secrets`
- `/admin/config`

**Widget Grid (Audit Logs):**
| Widget | Position | Size | Data Source | Render |
|--------|----------|------|-------------|--------|
| Audit Timeline | x:0 y:0 | w:12 h:16 | REST (keyset pagination) | Virtualized List |
| Filters | x:0 y:16 | w:12 h:4 | - | Form |
| Export | x:0 y:20 | w:12 h:4 | REST | Buttons |

---

# 📐 BÖLÜM 2: LAYOUT SİSTEMİ

## Genel Layout Blueprint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HEADER (60px, fixed)                                                       │
│  [Logo] [Operator|Analyst|DevOps|Infra|MLOps|Admin] [Search 🔍] [🔔] [👤]    │
├───────────┬─────────────────────────────────────────────────────────────────┤
│           │                                                                 │
│  SIDEBAR  │                    MIDDLE AREA                                  │
│ (280px)   │                    (12-col Responsive Grid)                     │
│ collapsible│                                                                │
│  to 64px  │  ┌────────────────────────────────────────────────────────┐    │
│           │  │                                                        │    │
│  Rol-bazlı│  │          ROL SPESİFİK İÇERİK                          │    │
│  dinamik  │  │                                                        │    │
│  menü     │  │          (Widget Grid with RxJS Streams)              │    │
│           │  │                                                        │    │
│           │  └────────────────────────────────────────────────────────┘    │
│           │                                                                 │
├───────────┴─────────────────────────────────────────────────────────────────┤
│  FOOTER (40px, non-sticky)                                                  │
│  [Build: abc123] [Model: v3.1] [Canary: 10%] [Env: production]             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Header Spesifikasyonu

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Yükseklik** | **60px** | Fixed High-Density |
| **Pozisyon** | Fixed | Sticky top, z-index: 1000 |
| **Sol İçerik** | Logo (48x48) | Superbet Genesis branding |
| **Orta İçerik** | **Role Tabs** | [Operator] [Analyst] [DevOps] [Infra] [MLOps] [Admin] |
| **Tab Davranışı** | Click → Route change | localStorage persist selected role |
| **Tab Styling** | Active: accent-primary bg | Inactive: transparent |
| **Orta-Sağ** | System Status | WS Status ●, Kafka Lag, p99 latency |
| **Sağ İçerik** | Hedge Now, New Simulation, Search, Alerts 🔔, User 👤 | Quick actions |
| **Search Kısayol** | `K` veya `/` | Global fuzzy search açar |

## Sidebar Spesifikasyonu

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Genişlik** | **280px** (collapsed: 64px) | Toggle button sol alt |
| **Pozisyon** | Fixed left | Dock, z-index: 999 |
| **Primary Nav** | **Role-specific menu** | Seçili role göre değişir |
| **Secondary Nav** | Nested Menus | Modül altında alt sayfalar |
| **Icons** | Lucide Icons | Consistent iconography |
| **States** | Active (indigo bg), Hover (elevated bg), Disabled (muted) | |
| **Animation** | 200ms ease-in-out | Menu transition on role change |
| **Mobile** | Overlay + hamburger | Slide-in panel |

### Role-Specific Sidebar Menus

**Operator:**
```
⚡ Signal Center (default)
⚽ Matches
  ├── Pre-match
  └── Live
🎯 Predictions
⚠️ Risk
```

**DevOps:**
```
📊 Metrics
📜 Logs ⭐
🔍 Traces
🔔 Alerts
💚 Health
```

**Infrastructure:**
```
☸️ Kubernetes
🔄 ArgoCD
💻 Terminal ⭐⭐⭐
⚙️ Config
```

**MLOps:**
```
🧪 Experiments
🏋️ Training
📦 Features
✅ Data Quality
```

**Admin:**
```
👥 Users
🚩 Feature Flags
📋 Audit Logs ⭐
🔐 Secrets
⚙️ System Config
```

## Middle Area

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Grid Sistemi** | 12-column | TailwindCSS grid-cols-12 |
| **Breakpoints** | 4K: 1920+, 2xl: 1536, xl: 1280, lg: 1024 | Desktop-first |
| **Card Yapısı** | shadow-lg, rounded-xl | Glassmorphism effect |
| **Background** | Deep Space (#0A0A0F) | Dark mode default |
| **Padding** | 24px | Consistent spacing |

## Responsive Design Considerations

> ⚠️ **NOT:** Bu blueprint **Desktop-first, Mobile-Responsive** yaklaşımıyla tasarlanmıştır.  
> Münazara direktifine göre: "Responsive tasarıma odaklanmayın, desktop-first"

| Breakpoint | Pixel | Öncelik | Davranış |
|------------|-------|---------|----------|
| **4K+** | 1920+ | P0 (Primary) | Tam genişlik, tüm widgetlar görünür |
| **2xl** | 1536 | P1 | Sidebar genişliği korunur |
| **xl** | 1280 | P1 | Minimum desteklenen masaüstü |
| **lg** | 1024 | P2 | Tablet landscape, sidebar collapse |
| **md** | 768 | P3 (Future) | Tablet portrait, hamburger menu |
| **sm** | 640 | P3 (Future) | Mobile, overlay sidebar |

**Mobile-Responsive (Future Work):**
- Touch-friendly button sizes (min 44x44px)
- Font-size scaling (rem tabanlı)
- Swipe gestures for role switching
- PWA support (offline-first)

---

# ⚡ BÖLÜM 3: REAL-TIME MİMARİSİ

## Veri Akışı Pipeline

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        GO BFF → FRONTEND AKIŞI                              │
└─────────────────────────────────────────────────────────────────────────────┘

  Go BFF (mTLS)              SharedWorker              Main Thread
       │                          │                          │
       │     Protobuf Binary      │                          │
       ├─────────────────────────►│                          │
       │     WebSocket Frame      │                          │
       │                          │                          │
       │                    ┌─────┴─────┐                    │
       │                    │  DECODE   │                    │
       │                    │ Protobuf  │                    │
       │                    └─────┬─────┘                    │
       │                          │                          │
       │                    ┌─────┴─────┐                    │
       │                    │  RxJS     │                    │
       │                    │  Routing  │                    │
       │                    │  (by type)│                    │
       │                    └─────┬─────┘                    │
       │                          │                          │
       │              ┌───────────┼───────────┐              │
       │              ▼           ▼           ▼              │
       │         alarms$      matches$    logs$              │
       │              │           │           │              │
       │              └───────────┼───────────┘              │
       │                          │  BroadcastChannel        │
       │                          ├─────────────────────────►│
       │                          │                          │
       │                          │                   ┌──────┴──────┐
       │                          │                   │ useSyncExternal
       │                          │                   │ Store       │
       │                          │                   └──────┬──────┘
       │                          │                          │
       │                          │                   ┌──────┴──────┐
       │                          │                   │ rAF Batch   │
       │                          │                   │ Render      │
       │                          │                   └─────────────┘
```

## WS Kanalları (v2.1 Güncel)

| Kanal | Topic | Payload | Purpose |
|-------|-------|---------|---------|
| `/ws/alarms` | risk.verified | CloudEvents (Protobuf) | VSNR/CAS signals |
| `/ws/matches` | prematch, live | TwinDelta (Protobuf) | Match updates |
| `/ws/broadcast` | broadcast.queue.priority | BroadcastData (Protobuf) | Broadcast status |
| `/ws/metrics` | Prometheus scrape | JSON | System metrics |
| `/ws/logs` | loki.stream | LogEntry (Protobuf) | **YENİ v2.1** Log streaming |
| `/ws/terminal` | terminal.{session_id} | TerminalData (Binary) | **YENİ v2.1** xterm.js data |
| `/ws/agents` | agents.status | AgentState (Protobuf) | HRL agent status |
| `/ws/circuit-breakers` | cb.status | CBState (Protobuf) | CB status updates |

---

# 📜 BÖLÜM 4: KRİTİK MODÜL - LOGS UI

## Münazara Kararı

> **OY BİRLİĞİ:** `auditTime(100)` + `windowCount(500)` + **EWMA fallback** pattern kabul edildi.
> - **Avantaj:** Son logu garanti eder, CPU %40 düşük, UI jank önler
> - **Burst Koruması:** windowCount(500) ile maksimum 500 log/batch
> - **EWMA Fallback:** Yüksek yoğunlukta otomatik geçiş (Hybrid pattern)
> - **Virtualization:** react-window ile 10k satır cap

## UI Tasarımı

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  LOGS                                                          [🔍] [⚙️]   │
├─────────────────────────────────────────────┬───────────────────────────────┤
│                                             │                               │
│  LOG STREAM (70%)                           │  SEARCH/FILTER (30%)          │
│  ──────────────────────────────────         │  ─────────────────────────    │
│                                             │                               │
│  [12:45:01.234] [INFO] [kafka-consumer]     │  Service: [Dropdown ▼]        │
│  Message received: match_id=ARS_LIV         │                               │
│                                             │  Level:                       │
│  [12:45:01.456] [WARN] [risk-manager]       │  [DEBUG] [INFO] [WARN] [ERROR]│
│  VaR threshold approaching: 4.2%            │                               │
│                                             │  Time Range:                  │
│  [12:45:01.789] [ERROR] [broadcast]         │  [Last 15m ▼] [Custom...]     │
│  Twitter rate limit hit, buffering          │                               │
│                                             │  Query:                        │
│  [12:45:02.012] [INFO] [edl-inference]      │  [Lucene query input...]      │
│  Prediction complete, τ=0.32                │                               │
│                                             │  [🔍 Search] [Clear]          │
│  ... (virtualized, infinite scroll)         │                               │
│                                             │                               │
├─────────────────────────────────────────────┴───────────────────────────────┤
│  📊 Stats: 1,234 logs/s | 45,678 total | Lag: 120ms | Buffer: 0            │
└─────────────────────────────────────────────────────────────────────────────┘
```

## RxJS Implementation (Kesinleşen)

```typescript
// streams/logs.ts
import { 
  auditTime, windowCount, mergeMap, toArray, 
  bufferTime, sampleTime, switchMap, distinctUntilKeyChanged,
  animationFrameScheduler, observeOn, scan, shareReplay 
} from 'rxjs/operators';
import { BehaviorSubject, interval } from 'rxjs';

// EWMA Rate Calculator (1 saniye penceresi)
class EWMARate {
  private alpha = 0.3; // Smoothing factor
  private rate = 0;
  private count = 0;
  
  update(batchSize: number): number {
    this.count += batchSize;
    return this.rate;
  }
  
  tick(): number {
    this.rate = this.alpha * this.count + (1 - this.alpha) * this.rate;
    this.count = 0;
    return this.rate;
  }
}

const ewma = new EWMARate();
const rate$ = new BehaviorSubject<number>(0);

// Her saniye EWMA hesapla
interval(1000).subscribe(() => rate$.next(ewma.tick()));

// Log streaming with EWMA-based Hybrid backpressure (NİHAİ KARAR)
export const logs$ = webSocket<LogEntry>('/ws/logs').pipe(
  // EWMA threshold'a göre dinamik strateji seçimi
  switchMap(log => rate$.pipe(
    switchMap(currentRate => {
      ewma.update(1);
      
      if (currentRate < 1000) {
        // Düşük yoğunluk: auditTime(100) + animationFrameScheduler
        return of(log).pipe(
          auditTime(100),
          observeOn(animationFrameScheduler)
        );
      } else {
        // Yüksek yoğunluk (≥1000/s): bufferTime + sampleTime + windowCount
        return of(log).pipe(
          bufferTime(200),
          sampleTime(100),
          windowCount(500),
          mergeMap(w$ => w$.pipe(toArray())),
          distinctUntilKeyChanged('offset')
        );
      }
    })
  )),
  
  // Virtualization için son 10k logu tut (react-window cap)
  scan((acc, batch) => {
    const logs = Array.isArray(batch) ? batch : [batch];
    return [...acc, ...logs].slice(-10000);
  }, [] as LogEntry[]),
  
  // Cross-tab paylaşım
  shareReplay({ bufferSize: 1, refCount: true })
);

// React hook with react-window integration
export function useLogs() {
  return useSyncExternalStore(
    callback => {
      const sub = logs$.subscribe(callback);
      return () => sub.unsubscribe();
    },
    () => logsStore.get()
  );
}
```

## Backfill REST Endpoint (Kesinleşen)

```typescript
// api/logs.ts
// Virtual table'da scroll edildiğinde eksik logları backfill et

interface BackfillRequest {
  since: number;     // lastOffset - eksik logların başlangıcı
  until?: number;    // Opsiyonel bitiş
  limit: number;     // Default: 500
}

async function backfillLogs(req: BackfillRequest): Promise<LogEntry[]> {
  const params = new URLSearchParams({
    since: req.since.toString(),
    limit: req.limit.toString(),
    ...(req.until && { until: req.until.toString() })
  });
  
  // Loki/ClickHouse'dan eksik logları çek
  const response = await fetch(`/api/logs/backfill?${params}`);
  return response.json();
}

// Kullanım: Virtual scroll'da gap tespit edildiğinde
const gap = detectGapInLogs(logs);
if (gap) {
  const missing = await backfillLogs({ since: gap.start, until: gap.end, limit: 500 });
  logsStore.merge(missing);
}
```

## Virtualization (react-window)

```tsx
// components/LogStream.tsx
import { FixedSizeList as List } from 'react-window';
import AutoSizer from 'react-virtualized-auto-sizer';

export const LogStream: React.FC = () => {
  const logs = useLogs();
  
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => {
    const log = logs[index];
    return (
      <div style={style} className={`log-row log-level-${log.level}`}>
        <span className="log-timestamp">{log.timestamp}</span>
        <span className="log-level">[{log.level}]</span>
        <span className="log-service">[{log.service}]</span>
        <span className="log-message">{log.message}</span>
      </div>
    );
  };
  
  return (
    <div className="log-stream-container">
      <AutoSizer>
        {({ height, width }) => (
          <List
            height={height}
            width={width}
            itemCount={logs.length}
            itemSize={24} // Her log satırı 24px
            overscanCount={20} // Scroll performansı için
          >
            {Row}
          </List>
        )}
      </AutoSizer>
    </div>
  );
};
```

## Log Level Styling

```css
.log-level-debug { color: var(--text-muted); }
.log-level-info { color: var(--accent-cyan); }
.log-level-warn { 
  color: var(--accent-warning); 
  background: rgba(245, 158, 11, 0.1);
}
.log-level-error { 
  color: var(--accent-danger); 
  background: rgba(239, 68, 68, 0.1);
  font-weight: 600;
}
.log-level-fatal { 
  color: #FF0000; 
  background: rgba(255, 0, 0, 0.2);
  font-weight: 700;
  animation: pulse 1s infinite;
}
```

---

# 💻 BÖLÜM 5: KRİTİK MODÜL - TERMINAL/SSH

## Münazara Kararı

> **OY BİRLİĞİ:** xterm.js + WebSocket + Kafka audit logging kabul edildi.
> - **Security:** mTLS + JWT + 15dk idle timeout + RBAC command allowlist
> - **Audit:** `audit.terminal.commands` topic, 180d TTL, command redaction
> - **Read-Only Mode:** Güvenli izleme için read-only toggle seçeneği

## UI Tasarımı

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  TERMINAL                              [🔒 Read-Only] [+ New Tab] [📋] [⚙️] │
├─────────────────────────────────────────────────────────────────────────────┤
│  [kafka-consumer-0] [risk-manager-1] [broadcast-0] [+]                      │
├─────────────────────────────────────────────────────────────────────────────┤
│  Pod: [kafka-consumer-0 ▼]  Namespace: [production ▼]  [🔄 Reconnect]      │
│  Mode: [● Interactive] [○ Read-Only]  ← Toggle ile güvenli görüntüleme     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  root@kafka-consumer-0:/app$ kubectl logs -f kafka-consumer                 │
│  [2026-01-04T14:30:00Z] INFO Starting consumer...                          │
│  [2026-01-04T14:30:01Z] INFO Connected to kafka:9092                       │
│  [2026-01-04T14:30:02Z] INFO Subscribed to topics: risk.verified           │
│  █                                                                          │
│                                                                             │
│                                                                             │
│                                                                             │
│                                                                             │
│                                                                             │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  Session: abc123 | Connected: 00:05:23 | Idle Timeout: 09:54               │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Security Implementation (Kesinleşen)

```go
// bff/terminal/handler.go
package terminal

import (
    "github.com/gorilla/websocket"
    "github.com/segmentio/kafka-go"
)

type TerminalSession struct {
    SessionID   string
    UserID      string
    Pod         string
    Namespace   string
    StartTime   time.Time
    LastActive  time.Time
    IdleTimeout time.Duration // 15 minutes
}

// Audit Kafka topic schema
type TerminalAuditEvent struct {
    UserID    string    `json:"user_id"`
    Timestamp int64     `json:"timestamp"`
    Pod       string    `json:"pod"`
    Namespace string    `json:"namespace"`
    Command   string    `json:"command"` // Redacted if sensitive
    SessionID string    `json:"session_id"`
    ExitCode  int       `json:"exit_code,omitempty"`
}

// Command redaction for sensitive commands
var sensitiveCommands = []string{"kubectl delete", "kubectl apply", "kubectl exec"}

func redactCommand(cmd string) string {
    for _, sensitive := range sensitiveCommands {
        if strings.HasPrefix(cmd, sensitive) {
            return sensitive + " <REDACTED>"
        }
    }
    return cmd
}

// RBAC command allowlist check
func isCommandAllowed(userRole string, command string) bool {
    allowlist := map[string][]string{
        "operator": {"kubectl get", "kubectl describe", "kubectl logs"},
        "devops":   {"kubectl get", "kubectl describe", "kubectl logs", "kubectl top"},
        "infra":    {"*"}, // Full access
        "admin":    {"*"},
    }
    // Check allowlist logic...
}
```

## Kafka Audit Topic Schema (Kesinleşen)

```yaml
# kafka/topics/audit.terminal.commands.yaml
topic: audit.terminal.commands
partitions: 12
replication_factor: 3
retention.ms: 15552000000  # 180 days
config:
  cleanup.policy: delete
  compression.type: lz4
  
schema:
  type: record
  name: TerminalAuditEvent
  fields:
    - name: user_id
      type: string
    - name: timestamp
      type: long
      logicalType: timestamp-millis
    - name: pod
      type: string
    - name: namespace
      type: string
    - name: command
      type: string
      doc: "Redacted for kubectl delete/apply/exec"
    - name: session_id
      type: string
    - name: exit_code
      type: ["null", "int"]
```

---

# 🤖 BÖLÜM 6: KRİTİK MODÜL - ML PLATFORM

## Münazara Kararı

> **OY BİRLİĞİ:** Phase-1 = iframe + SSO token injection kabul edildi.
> - **Token TTL:** 300s (5 dakika)
> - **Token Refresh:** T-45s (255 saniyede bir)
> - **CSP:** `frame-ancestors 'self'`
> - **Phase-2 (Roadmap):** BFF-normalize widget'lar

## iframe + SSO Implementation (Kesinleşen)

```tsx
// components/mlflow/MLflowEmbed.tsx
import { useEffect, useRef, useState } from 'react';

interface MLflowEmbedProps {
  experimentId?: string;
}

export const MLflowEmbed: React.FC<MLflowEmbedProps> = ({ experimentId }) => {
  const iframeRef = useRef<HTMLIFrameElement>(null);
  const [token, setToken] = useState<string>('');
  const [expiresAt, setExpiresAt] = useState<number>(0);

  // Initial token fetch
  useEffect(() => {
    fetchToken();
  }, []);

  // Token refresh at T-45s
  useEffect(() => {
    const refreshTime = expiresAt - 45000; // 45s before expiry
    const now = Date.now();
    
    if (refreshTime > now) {
      const timeout = setTimeout(fetchToken, refreshTime - now);
      return () => clearTimeout(timeout);
    }
  }, [expiresAt]);

  async function fetchToken() {
    const response = await fetch('/api/mlflow/token');
    const data = await response.json();
    setToken(data.token);
    setExpiresAt(Date.now() + data.expires_in * 1000);
    
    // PostMessage to iframe for token update
    if (iframeRef.current?.contentWindow) {
      iframeRef.current.contentWindow.postMessage(
        { type: 'TOKEN_UPDATE', token: data.token },
        'https://mlflow.superbet.internal'
      );
    }
  }

  const mlflowUrl = `https://mlflow.superbet.internal:5000/${
    experimentId ? `experiments/${experimentId}` : ''
  }?access_token=${token}`;

  return (
    <div className="mlflow-container">
      <iframe
        ref={iframeRef}
        src={mlflowUrl}
        className="w-full h-full border-0"
        sandbox="allow-scripts allow-same-origin allow-forms"
        title="MLflow UI"
      />
      
      {/* Overlay action buttons */}
      <div className="absolute bottom-4 right-4 flex gap-2">
        <button className="btn-primary" onClick={handlePromoteToProduction}>
          🚀 Promote to Production
        </button>
        <button className="btn-secondary" onClick={handleCompareRuns}>
          📊 Compare Runs
        </button>
      </div>
    </div>
  );
};
```

## BFF Token Endpoint

```go
// bff/mlflow/token.go
package mlflow

type TokenResponse struct {
    Token     string `json:"token"`
    ExpiresIn int    `json:"expires_in"` // 300 seconds
}

func TokenHandler(w http.ResponseWriter, r *http.Request) {
    // Validate user JWT from request
    userClaims := extractJWTClaims(r)
    
    // Generate MLflow-specific short-lived token
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
        "sub": userClaims.UserID,
        "role": userClaims.Role,
        "exp": time.Now().Add(5 * time.Minute).Unix(),
        "aud": "mlflow",
    })
    
    tokenString, _ := token.SignedString(mlflowSecretKey)
    
    json.NewEncoder(w).Encode(TokenResponse{
        Token:     tokenString,
        ExpiresIn: 300,
    })
}
```

---

# 🔐 BÖLÜM 7: RBAC VE GÜVENLİK

## Permission Matrix (Kesinleşen)

| Modül | Operator | Analyst | DevOps | Infra | MLOps | Admin |
|-------|----------|---------|--------|-------|-------|-------|
| **Signal Center** | ✅ Read | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **Matches** | ✅ Read | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **Predictions** | ✅ Read | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **Risk** | ✅ Read | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **Strategies** | ❌ | ✅ Read/Write | ❌ | ❌ | ❌ | ✅ Full |
| **Backtest** | ❌ | ✅ Read/Write | ❌ | ❌ | ❌ | ✅ Full |
| **Reports** | ❌ | ✅ Read/Export | ❌ | ❌ | ❌ | ✅ Full |
| **Metrics** | ❌ | ❌ | ✅ Read | ✅ Read | ❌ | ✅ Full |
| **Logs** | ❌ | ❌ | ✅ Read/Filter | ✅ Read | ❌ | ✅ Full |
| **Traces** | ❌ | ❌ | ✅ Read | ❌ | ❌ | ✅ Full |
| **Alerts** | ❌ | ❌ | ✅ Read/Ack | ❌ | ❌ | ✅ Full |
| **Kubernetes** | ❌ | ❌ | ❌ | ✅ Read/Write | ❌ | ✅ Full |
| **ArgoCD** | ❌ | ❌ | ❌ | ✅ Read/Rollback | ❌ | ✅ Full |
| **Terminal/SSH** | ❌ | ❌ | ❌ | ✅ Full (allowlist) | ❌ | ✅ Full |
| **Config** | ❌ | ❌ | ❌ | ✅ Read/Write | ❌ | ✅ Full |
| **ML Experiments** | ❌ | ❌ | ❌ | ❌ | ✅ Read | ✅ Full |
| **ML Training** | ❌ | ❌ | ❌ | ❌ | ✅ Read/Cancel | ✅ Full |
| **ML Features** | ❌ | ❌ | ❌ | ❌ | ✅ Read | ✅ Full |
| **ML Promote** | ❌ | ❌ | ❌ | ❌ | ✅ Request | ✅ Approve |
| **Users** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ Full |
| **Audit Logs** | ❌ | ❌ | ✅ Read | ✅ Read | ✅ Read | ✅ Full |
| **Secrets** | ❌ | ❌ | ❌ | ✅ Read (masked) | ❌ | ✅ Full |
| **System Config** | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ Full |

## Frontend RBAC Enforcement

```tsx
// hooks/usePermission.ts
import { useAuth } from './useAuth';

const PERMISSIONS: Record<Role, Record<string, Permission[]>> = {
  operator: {
    '/operator/*': ['read'],
  },
  analyst: {
    '/analyst/*': ['read', 'write'],
  },
  devops: {
    '/devops/*': ['read'],
    '/devops/alerts': ['read', 'acknowledge'],
  },
  infra: {
    '/infra/*': ['read', 'write'],
    '/infra/terminal': ['read', 'write', 'execute'],
  },
  mlops: {
    '/mlops/*': ['read'],
    '/mlops/experiments': ['read', 'promote_request'],
  },
  admin: {
    '/*': ['read', 'write', 'delete', 'admin'],
  },
};

export function usePermission(path: string, action: Permission): boolean {
  const { user } = useAuth();
  if (!user) return false;
  
  const rolePermissions = PERMISSIONS[user.role];
  // Check path match and permission...
}

// Usage in components
export const TerminalPage = () => {
  const canExecute = usePermission('/infra/terminal', 'execute');
  
  if (!canExecute) {
    return <AccessDenied />;
  }
  
  return <Terminal />;
};
```

---

# 📋 BÖLÜM 8: AUDIT LOG RETENTION

## Münazara Kararı

> **OY BİRLİĞİ:** UI 90d (ClickHouse Hot) + Archive 180d (S3 Cold) kabul edildi.
> - **Pagination:** Keyset pagination (timestamp, id)
> - **Load More:** before_ts, limit=200

## Audit Log UI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  AUDIT LOGS                                          [📅 90 Days] [Export] │
├─────────────────────────────────────────────────────────────────────────────┤
│  Filters: [User ▼] [Action ▼] [Resource ▼] [🔍 Search...]                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─ Jan 04, 2026 ─────────────────────────────────────────────────────────┐│
│  │ 14:30:45  admin@superbet.com  PROMOTE  model/edl-v3.2        SUCCESS   ││
│  │ 14:28:12  devops@superbet.com ROLLBACK argocd/kafka-consumer WARNING   ││
│  │ 14:15:03  infra@superbet.com  EXECUTE  terminal/kafka-pod-0  SUCCESS   ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│  ┌─ Jan 03, 2026 ─────────────────────────────────────────────────────────┐│
│  │ 23:45:00  system              BACKUP   database/audit-logs   SUCCESS   ││
│  │ ...                                                                     ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│                         [Load More (200 remaining)]                         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  📁 Archive (180d) available via [Request Export] button                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Keyset Pagination Implementation

```typescript
// api/audit.ts
interface AuditLogQuery {
  before_ts?: number;  // Keyset: timestamp
  before_id?: string;  // Keyset: id
  limit: number;       // Default: 200
  user?: string;
  action?: string;
  resource?: string;
}

async function fetchAuditLogs(query: AuditLogQuery) {
  const params = new URLSearchParams({
    limit: query.limit.toString(),
    ...(query.before_ts && { before_ts: query.before_ts.toString() }),
    ...(query.before_id && { before_id: query.before_id }),
  });
  
  // ClickHouse query uses keyset for efficient pagination
  // SELECT * FROM audit_logs 
  // WHERE (timestamp, id) < (:before_ts, :before_id)
  // ORDER BY timestamp DESC, id DESC
  // LIMIT :limit
  
  return fetch(`/api/audit/logs?${params}`).then(r => r.json());
}
```

---

# 🎨 BÖLÜM 9: TEMA SİSTEMİ

## Design Tokens (Updated v2.1)

```css
:root {
  /* Background (Dark Mode Default - Deep Space) */
  --bg-primary: #0A0A0F;
  --bg-secondary: #12121A;
  --bg-tertiary: #1A1A25;
  --bg-elevated: #22222E;
  
  /* Accent Colors (Color-blind safe) */
  --accent-primary: #4F46E5;    /* Indigo - Primary actions */
  --accent-cyan: #00D4FF;        /* Signals, WS connected */
  --accent-success: #10B981;     /* Success, profit */
  --accent-warning: #F59E0B;     /* Warning, CB HALF_OPEN */
  --accent-danger: #EF4444;      /* Danger, CB OPEN */
  --accent-purple: #9966FF;      /* AI/ML indicators */
  
  /* Terminal Colors */
  --terminal-bg: #0D0D0D;
  --terminal-fg: #00FF00;
  --terminal-cursor: #00FF00;
  --terminal-selection: rgba(0, 255, 0, 0.3);
  
  /* Log Level Colors */
  --log-debug: #6B7280;
  --log-info: #00D4FF;
  --log-warn: #F59E0B;
  --log-error: #EF4444;
  --log-fatal: #FF0000;
  
  /* Text */
  --text-primary: #FFFFFF;
  --text-secondary: #A0A0B0;
  --text-muted: #606070;
  
  /* Spacing (4px grid) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  
  /* Typography */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', 'Consolas', monospace;
  
  /* Font Sizes */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 24px;
  --text-2xl: 32px;
}
```

---

# 🚀 BÖLÜM 10: DEPLOYMENT VE SONRAKI ADIMLAR

## Helm Chart Yapısı (v2.1 Updated)

```yaml
# infra/helm/charts/frontend-shell/values.yaml
replicaCount: 2

image:
  repository: superbet/frontend-shell
  tag: v2.1

bff:
  enabled: true
  image: superbet/go-bff:v2.1
  port: 8080
  kafka:
    brokers: ["kafka:9092"]
    topics:
      - risk.verified
      - prematch
      - live
      - broadcast.queue.priority
      - loki.stream           # YENİ v2.1
      - audit.terminal.commands  # YENİ v2.1
  redis:
    host: redis-cluster:6379
  terminal:
    enabled: true
    idleTimeout: 15m
    auditTopic: audit.terminal.commands
  mlflow:
    enabled: true
    tokenTTL: 300

modules:
  - name: signal-center
  - name: matches
  - name: risk
  - name: logs         # YENİ v2.1
  - name: terminal     # YENİ v2.1
  - name: mlops        # YENİ v2.1

rbac:
  enabled: true
  roles:
    - operator
    - analyst
    - devops
    - infra
    - mlops
    - admin
```

## Sonraki Adımlar

| Adım | Öncelik | Tahmini Süre | Sorumlu |
|------|---------|--------------|---------|
| Shell App + Role Tabs | P0 | 3 gün | Frontend |
| Signal Center Module | P0 | 2 gün | Frontend |
| Go BFF Log Streaming | P0 | 2 gün | Backend |
| Logs UI Component | P0 | 3 gün | Frontend |
| Terminal/SSH Module | P1 | 4 gün | Full-stack |
| RBAC Middleware | P1 | 2 gün | Backend |
| MLflow iframe Integration | P1 | 2 gün | Frontend |
| Audit Log UI | P2 | 2 gün | Frontend |
| Admin Dashboard | P2 | 3 gün | Frontend |

---

# 📊 NİHAİ KARAR MATRİSİ

| Kategori | Karar | Teknoloji | Gerekçe |
|----------|-------|-----------|---------|
| **Mimari** | Rol Bazlı Modüler SPA | Webpack MF + Role Routing | 6 rol için optimize |
| **BFF** | Go BFF | Go + Gorilla WS + xterm.js proxy | Terminal + Log streaming |
| **State** | RxJS Streams | RxJS 7 | Unified stream management |
| **Transport** | Protobuf | protobuf.js | %60 payload azalma |
| **Role Tabs** | Header Tabs | LocalStorage persist | Hızlı geçiş |
| **Log Pattern** | auditTime(100) + windowCount(500) + **EWMA fallback** | RxJS 7, react-window | Burst korumalı, Hybrid |
| **Log Virtualization** | 10k satır cap | react-window + AutoSizer | Performans |
| **Log Backfill** | REST endpoint (since=lastOffset) | Loki/ClickHouse | Gap doldurmak için |
| **Terminal** | xterm.js + WS + **Read-Only toggle** | Go proxy + Kafka audit | Güvenli SSH |
| **ML Embed** | iframe + SSO + postMessage | JWT 300s, T-45s refresh, CSP | Phase-1 MVP |
| **Audit Retention** | 90d UI / 180d Archive | ClickHouse + S3 + keyset pagination | Compliance |
| **RBAC** | 6 Rol Matrix | JWT claims + route guards | Granular control |


---

# 📋 NİHAİ İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Toplam Rol** | 6 (Operator, Analyst, DevOps, Infra, MLOps, Admin) |
| **Toplam Modül** | 20+ (rol bazlı gruplandırılmış) |
| **Widget Tipleri** | 25+ |
| **WS Kanalları** | 8 |
| **RxJS Streams** | 10+ (domain bazlı) |
| **API Endpoints** | 50+ |
| **Münazara Turları** | 5 Tur (2 oturum) |
| **Oylama Kararları** | 4 (Log, ML, Retention, Terminal) |
| **Panel Katılımcı** | 10 LLM |

---

# 🔧 EK BÖLÜM A: TEKNİK DETAYLAR (v2.0'dan)

> ⚠️ **NOT:** Bu bölüm FRONTEND_ARCHITECTURE_v2.0.md'den taşınmıştır.  
> v2.1 UI şemasını, v2.0 teknik altyapısını içerir. Tek referans dosyası olarak kullanılabilir.

---

## A.1 SharedWorker Implementasyonu

```typescript
// worker/shared.ts
const sockets = new Map<string, WebSocket>();
const channel = new BroadcastChannel('superbet-realtime');

self.onconnect = (e: MessageEvent) => {
  const port = e.ports[0];
  
  // Tek WS bağlantısı - tüm sekmeler paylaşır
  if (!sockets.has('main')) {
    const ws = new WebSocket('wss://bff.superbet:8080/ws/alarms');
    ws.binaryType = 'arraybuffer';
    
    ws.onmessage = (event) => {
      // BroadcastChannel ile tüm sekmelere yayınla
      channel.postMessage(event.data);
    };
    
    ws.onclose = () => {
      sockets.delete('main');
      // Auto-reconnect with exponential backoff
      setTimeout(() => reconnect(), 1000);
    };
    
    sockets.set('main', ws);
  }
  
  port.start();
};

// Reconnection logic
function reconnect(attempt = 0) {
  const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
  setTimeout(() => {
    try {
      const ws = new WebSocket('wss://bff.superbet:8080/ws/alarms');
      sockets.set('main', ws);
    } catch {
      reconnect(attempt + 1);
    }
  }, delay);
}
```

---

## A.2 Domain-Specific RxJS Streams

```typescript
// streams/index.ts
import { Subject, BehaviorSubject } from 'rxjs';
import { bufferTime, shareReplay, map, filter, throttleTime } from 'rxjs/operators';
import { decode } from 'protobufjs';

// SharedWorker'dan gelen raw binary
const rawBinary$ = new Subject<ArrayBuffer>();

// BroadcastChannel listener
const channel = new BroadcastChannel('superbet-realtime');
channel.onmessage = (e) => rawBinary$.next(e.data);

// Protobuf decode
const decoded$ = rawBinary$.pipe(
  map(buffer => decode(TwinDelta, new Uint8Array(buffer))),
  shareReplay({ bufferSize: 50, windowTime: 60000 })
);

// Domain streams with FULL backpressure strategy
export const alarms$ = decoded$.pipe(
  filter(msg => msg.type === 'alarm'),
  bufferTime(250),           // Batch 250ms
  map(batch => batch.slice(-100)), // Son 100 alarm
);

export const matches$ = decoded$.pipe(
  filter(msg => msg.type === 'match'),
  map(groupByMatchId),
  throttleTime(100),         // Match updates throttle
);

export const broadcast$ = decoded$.pipe(
  filter(msg => msg.type === 'broadcast'),
  map(event => ({
    ...event,
    isFiltered: event.metrics.confidence > 0.65 &&
                event.metrics.vsnr > 2.2 &&
                event.metrics.cas > 1.0,
    // τ > 0.4 → eylem kilidi
    isLocked: event.metrics.uncertainty > 0.4
  })),
);

export const agents$ = decoded$.pipe(
  filter(msg => msg.type === 'agent_status'),
);

export const circuitBreakers$ = decoded$.pipe(
  filter(msg => msg.type === 'cb_status'),
);
```

---

## A.3 SWR Cache Stratejisi

```typescript
// cache/swr.ts
import { BehaviorSubject, merge, of, from, Observable } from 'rxjs';
import { switchMap, tap } from 'rxjs/operators';
import { openDB } from 'idb';

// IndexedDB setup
const dbPromise = openDB('superbet-cache', 1, {
  upgrade(db) {
    db.createObjectStore('cache');
  },
});

class SWRCache<T> {
  private cache$ = new BehaviorSubject<T | null>(null);
  private staleTime = 30000; // 30s
  private lastFetch = 0;
  
  async get(key: string, fetcher: () => Promise<T>): Promise<Observable<T>> {
    const cached = this.cache$.value;
    const isStale = Date.now() - this.lastFetch > this.staleTime;
    
    // Try IndexedDB first
    if (!cached) {
      const db = await dbPromise;
      const idbCached = await db.get('cache', key);
      if (idbCached) {
        this.cache$.next(idbCached);
      }
    }
    
    if (cached && !isStale) {
      return of(cached);
    }
    
    // Stale-while-revalidate: Önce stale veriyi göster, arka planda yenile
    const fresh$ = from(fetcher()).pipe(
      tap(async data => {
        this.cache$.next(data);
        this.lastFetch = Date.now();
        // IndexedDB'ye persist
        const db = await dbPromise;
        await db.put('cache', data, key);
      })
    );
    
    if (cached) {
      // WS event → UI update → arka planda REST doğrulama
      return merge(
        of(cached),           // Hemen stale veriyi göster
        fresh$                // Arka planda yenile
      );
    }
    
    return fresh$;
  }
}

// Usage
const matchCache = new SWRCache<Match>();
const predictionCache = new SWRCache<Prediction>();
```

---

## A.4 Bileşen Kütüphanesi

| Bileşen | Amaç | Render | Kütüphane |
|---------|------|--------|-----------|
| **Card** | Widget container | DOM/CSS | - |
| **Gauge** | KPI göstergeleri (ROI, Sharpe) | SVG | - |
| **Donut** | CB Status dağılımı | SVG | - |
| **Sparkline** | Trend mini grafikler | SVG | - |
| **Virtual Table** | Büyük listeler (Matches, Logs) | DOM + windowing | react-window |
| **Heatmap** | EDL Uncertainty, VSNR/CAS | WebGL (Canvas) | Deck.gl |
| **Timeline** | Match xG alan grafiği | SVG | - |
| **AlertBanner** | CB OPEN sarı şerit | DOM | - |
| **Terminal** | SSH/Pod exec | xterm.js | xterm.js + xterm-addon-fit |
| **LogViewer** | Real-time log stream | Virtualized List | react-window |
| **MLflowEmbed** | MLflow UI embedding | iframe | - |

---

## A.5 Protobuf Schema Tanımları

```protobuf
// proto/superbet.proto

syntax = "proto3";

package superbet;

message TwinDelta {
  string type = 1;  // alarm, match, broadcast, agent_status, cb_status
  bytes payload = 2;
  int64 timestamp = 3;
}

message AlarmEvent {
  string match_id = 1;
  double vsnr = 2;
  double cas = 3;
  double confidence = 4;
  double uncertainty = 5;  // EDL τ
  string prediction = 6;   // H, D, A
  double odds = 7;
  int64 kickoff = 8;
}

message MatchUpdate {
  string match_id = 1;
  string phase = 2;        // prematch, live
  int32 home_score = 3;
  int32 away_score = 4;
  double xg_home = 5;
  double xg_away = 6;
  repeated OddsHistory odds_history = 7;
}

message OddsHistory {
  int64 timestamp = 1;
  double home_odds = 2;
  double draw_odds = 3;
  double away_odds = 4;
}

message BroadcastEvent {
  string id = 1;
  string match_id = 2;
  string platform = 3;     // twitter, telegram, android
  bool sent = 4;
  string formatted_text = 5;
  double priority_score = 6;
}

message AgentStatus {
  string agent_type = 1;   // prematch, live
  double gamma = 2;        // Liderlik/Eşgüdüm modu
  string current_match = 3;
}

message CircuitBreakerStatus {
  string name = 1;
  string state = 2;        // CLOSED, OPEN, HALF_OPEN
  int32 failure_count = 3;
  int64 last_failure = 4;
}

message LogEntry {
  int64 timestamp = 1;
  string level = 2;        // DEBUG, INFO, WARN, ERROR, FATAL
  string service = 3;
  string message = 4;
  int64 offset = 5;
}

message TerminalData {
  string session_id = 1;
  bytes data = 2;          // xterm.js binary data
}
```

---

## A.6 CI/CD Pipeline

```yaml
# .github/workflows/frontend-deploy.yml
name: Frontend Deploy

on:
  push:
    branches: [main]
    paths: ['frontend/**']

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Build Shell
        run: |
          cd frontend/shell
          npm ci
          npm run build
          
      - name: Build Modules
        run: |
          for module in signal-center matches risk logs terminal mlops admin; do
            cd frontend/$module
            npm ci && npm run build
            cd ..
          done
          
      - name: Push to CDN
        run: aws s3 sync dist/ s3://superbet-cdn/frontend/
        
      - name: Invalidate CloudFront
        run: aws cloudfront create-invalidation --distribution-id $CF_DIST --paths "/*"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Update Helm
        run: |
          yq e '.frontend.image.tag = "${{ github.sha }}"' -i helm/values.yaml
          git commit -m "Deploy frontend ${{ github.sha }}"
          git push

  e2e-test:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Run Playwright Tests
        run: |
          cd frontend/e2e
          npx playwright test --project=chromium
```

---

## A.7 Module Federation Yapılandırması

```javascript
// frontend/shell/webpack.config.js
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        'signal-center': 'signalCenter@https://cdn.superbet/signal-center/remoteEntry.js',
        'matches': 'matches@https://cdn.superbet/matches/remoteEntry.js',
        'risk': 'risk@https://cdn.superbet/risk/remoteEntry.js',
        'logs': 'logs@https://cdn.superbet/logs/remoteEntry.js',
        'terminal': 'terminal@https://cdn.superbet/terminal/remoteEntry.js',
        'mlops': 'mlops@https://cdn.superbet/mlops/remoteEntry.js',
        'admin': 'admin@https://cdn.superbet/admin/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.2.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.2.0' },
        rxjs: { singleton: true, requiredVersion: '^7.8.0' },
        'react-window': { singleton: true },
        'xterm': { singleton: true },
      },
    }),
  ],
};
```

---

## A.8 BFF Entegrasyon Şeması

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SİSTEM ENTEGRASYON HARİTASI                          │
└─────────────────────────────────────────────────────────────────────────────┘

  Browser (Modüler SPA)                     Go BFF (:8080)
  ─────────────────────                     ─────────────────
  │                                         │
  │ WebSocket (Binary/Protobuf)             │ ← mTLS Auth
  ├────────────────────────────────────────►│ ← Vault Secrets
  │                                         │
  │                                         ├── Kafka Consumer
  │                                         │   ├── risk.verified
  │                                         │   ├── prematch, live
  │                                         │   ├── broadcast.queue.priority
  │                                         │   ├── loki.stream (logs)
  │                                         │   └── audit.terminal.commands
  │                                         │
  │                                         ├── Redis (Rate Limits)
  │                                         │   └── broadcast:limits:*
  │                                         │
  │                                         ├── Terminal Proxy
  │                                         │   └── kubectl exec → xterm.js
  │                                         │
  │                                         └── Prometheus Query
  │                                             └── /api/v1/query
  │
  │ REST API (JSON)
  ├────────────────────────────────────────►│
  │ GET /api/matches/:id                    │ ← Feast Features
  │ GET /api/predictions                    │ ← ClickHouse
  │ POST /api/hedge                         │ ← Risk Actions
  │ GET /api/logs/backfill                  │ ← Loki (YENİ v2.1)
  │ GET /api/mlflow/token                   │ ← Vault (YENİ v2.1)
  │ GET /api/audit                          │ ← ClickHouse (YENİ v2.1)

  Shell App                          Remote Modules (Webpack MF)
  ─────────                          ──────────────────────────
  │                                  │
  │ loadRemote('./signal-center')    │
  ├─────────────────────────────────►│ signal-center (CDN)
  │                                  │
  │ loadRemote('./matches')          │
  ├─────────────────────────────────►│ matches (CDN)
  │                                  │
  │ loadRemote('./logs')             │
  ├─────────────────────────────────►│ logs (CDN) ← YENİ v2.1
  │                                  │
  │ loadRemote('./terminal')         │
  ├─────────────────────────────────►│ terminal (CDN) ← YENİ v2.1
  │                                  │
  │ loadRemote('./mlops')            │
  ├─────────────────────────────────►│ mlops (CDN) ← YENİ v2.1
```

---

**Kaynak:** TETRA AI Panel Full-Stack UI Münazarası (2 oturum, 5 tur)  
**Versiyon:** v2.1 GENESIS UI (Teknik Detaylar v2.0'dan birleştirildi)  
**Tarih:** 04.01.2026  
**Referans:** bettingenesis-v3.1.md + BROADCAST_LAYER_v3.1.md + FRONTEND_ARCHITECTURE_v2.0.md  
**Misyon:** Role-Based Command Center + Full-Stack UI Coverage + Complete Technical Specification

