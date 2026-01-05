# 🎨 SUPERBET GENESIS v3.1 - FULL-STACK UI TASARIM MÜNAZARASI
## Rol Bazlı Command Center + Infrastructure + ML Platform

**Versiyon:** v2.0  
**Tarih:** 04.01.2026  
**Amaç:** Tüm sistemin UI temsili - Operator'den Admin'e, DevOps'tan MLOps'a

---

## 📋 MÜNAZARA KURALLARI

Bu münazara **oy birliği sağlanana kadar** devam edecektir:

- **Tur 1:** Her katılımcı kendi UI/UX vizyonunu sunacak
- **Tur 2+:** Ortak paydalar belirlenecek, çatışmalar tartışılacak
- **Final:** Oy birliği ile nihai UI blueprint onaylanacak

**ÖNEMLI:** 
- Production-ready kod yazmayın!
- Kavramsal UI/UX tasarımı ve blueprint üzerine odaklanın
- Küçük code snippet'ler mantığı açıklamak için kullanılabilir
- **MASAÜSTÜ ODAKLI** - Responsive tasarıma odaklanmayın, desktop-first

---

## 🎯 GÖREV TANIMI

### REFERANS DOKÜMANLAR

Size 3 referans doküman sunuldu:

1. **bettingenesis-v3.1.md** - Backend mimarisi (Veri katmanı, AI/ML, Risk, Observability)
2. **BROADCAST_LAYER_v3.1.md** - Broadcast katmanı (Twitter, Telegram, Android push)
3. **FRONTEND_ARCHITECTURE_v2.0.md** - Mevcut frontend teknik mimarisi

### PROBLEM

Mevcut frontend mimarisi **Operator odaklı** ve **7 modül** içeriyor. Ancak:

**❌ EKSİK OLANLAR:**
- Log sistemi UI yok (ELK/Loki)
- SSH/Terminal erişimi yok
- ML Platform yönetimi yok (MLflow, Ray, Feast, Optuna)
- Infrastructure kontrolü yok (K8s, ArgoCD)
- Tam RBAC ve Audit log yok
- Backend'deki birçok kavram UI'da temsil edilmiyor

### GÖREV

**Tüm sistemi kapsayan, rol bazlı, enterprise-grade UI tasarımı** oluşturun.

---

## 🏗️ MEVCUT vs OLMASI GEREKEN

### Mevcut UI (7 Modül - Yetersiz):
```
├── Signal Center     → Operator
├── Matches           → Operator
├── Predictions       → Operator
├── Risk              → Operator
├── Strategies        → Analyst
├── Monitoring        → DevOps (sınırlı)
└── Settings          → Admin (basit)
```

### Olması Gereken UI (Rol Bazlı - Tam Sistem):

```
┌─────────────────────────────────────────────────────────────────────────┐
│  SUPERBET GENESIS v3.1 - COMMAND CENTER                                 │
├─────────────────────────────────────────────────────────────────────────┤
│  [Operator] [Analyst] [DevOps] [Infra] [MLOps] [Admin]  ← Rol Sekmeleri │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Seçili role göre Sidebar ve Content dinamik olarak değişir             │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 HER ROL İÇİN MODÜLLER

### 1. 📊 OPERATOR DASHBOARD
```
├── Signal Center
│   ├── VSNR/CAS Alarm Feed (real-time)
│   ├── EDL Uncertainty Heatmap
│   └── γ Gamma Rejim Göstergesi
├── Matches
│   ├── Pre-match List
│   ├── Live Matches
│   ├── Match Detail (Timeline, xG, Odds)
│   └── Pre→Live Handover Status
├── Predictions
│   ├── HRL Önerileri
│   ├── Kupon Kombinasyonları (Trixie, Yankee, etc.)
│   ├── Kelly Pay Dağılımı
│   └── Broadcast Yayın Durumu
└── Risk
    ├── VaR/CVaR/MaxDD/Sharpe Gauges
    ├── Risk Limitleri (Günlük/Haftalık)
    └── Circuit Breaker Status Matrix
```

### 2. 📈 ANALYST DASHBOARD
```
├── Strategies
│   ├── UCB Manager ROI Grafiği
│   ├── Strateji Ağırlıkları (10+ strateji)
│   ├── BCD Rejim Algılama (p_BCD trend)
│   └── Mode Matrix (Liderlik/Eşgüdüm/Nötr)
├── Backtest (YENİ)
│   ├── Tarihsel Performans Analizi
│   ├── Strateji Karşılaştırma
│   └── Drawdown Analizi
└── Reports (YENİ)
    ├── Günlük Özet
    ├── Haftalık Rapor
    └── Export (PDF/CSV)
```

### 3. 🔧 DEVOPS DASHBOARD
```
├── Metrics
│   ├── Prometheus Dashboard (embed veya mirror)
│   ├── Business KPI'lar (ROI, Win Rate)
│   └── SLO Dashboard (p99 latency, uptime)
├── Logs (YENİ - KRİTİK) ⭐
│   ├── Real-time Log Stream (tail -f)
│   ├── Search/Filter (service, level, time range)
│   ├── Log Levels (DEBUG/INFO/WARN/ERROR/FATAL)
│   └── Log Aggregation (ELK veya Loki)
├── Traces
│   ├── Jaeger UI (embed veya link)
│   ├── Span Visualization
│   └── Latency Distribution
├── Alerts (YENİ)
│   ├── Active Alerts List
│   ├── Alert History
│   ├── Acknowledge/Silence Actions
│   └── PagerDuty/Slack Integration Status
└── Health
    ├── Service Status (UP/DOWN/DEGRADED)
    ├── Uptime Statistics
    └── Dependency Graph
```

### 4. 🏗️ INFRASTRUCTURE DASHBOARD (YENİ) ⭐
```
├── Kubernetes
│   ├── Cluster Overview
│   ├── Pods Status (Running/Pending/Failed)
│   ├── Deployments List
│   ├── HPA Status (current/min/max replicas)
│   ├── Resource Usage (CPU/Memory per pod)
│   └── Restart Counts
├── ArgoCD
│   ├── Applications List
│   ├── Sync Status (Synced/OutOfSync/Unknown)
│   ├── Canary Deployment Progress (%10 → %100)
│   ├── Rollback Button (one-click)
│   └── Git Commit History
├── Terminal/SSH (YENİ - KRİTİK) ⭐⭐⭐
│   ├── Pod Selection Dropdown
│   ├── Interactive Shell (xterm.js + WebSocket)
│   ├── Command History
│   ├── Multiple Tab Support
│   └── Log Streaming (kubectl logs -f)
└── Config
    ├── Consul/Etcd Key-Value Browser
    ├── Config Diff View
    ├── Edit & Apply (with audit)
    └── Vault Secrets (masked view)
```

### 5. 🤖 ML PLATFORM DASHBOARD (YENİ) ⭐
```
├── Experiments (MLflow)
│   ├── Experiment List
│   ├── Run Comparison (metrics side-by-side)
│   ├── Model Registry (versions, stages)
│   ├── Artifacts Browser
│   └── Promote to Production Button
├── Training (Ray.io)
│   ├── Active Jobs List
│   ├── Job Details (progress, logs)
│   ├── GPU/CPU Usage per Job
│   ├── Queue Status
│   └── Cancel/Restart Actions
├── Features (Feast)
│   ├── Feature Store Browser
│   ├── Feature Definitions
│   ├── Freshness Status (TTL, last update)
│   ├── Usage Statistics
│   └── Feature Lineage
├── Data Quality (Great Expectations)
│   ├── Validation Run History
│   ├── Failed Checks List
│   ├── Data Profiles
│   ├── Expectation Suite Editor
│   └── Alert on Failure
└── Hyperparameters (Optuna)
    ├── Study List
    ├── Trial History
    ├── Best Parameters View
    ├── Optimization Progress Chart
    └── Parameter Importance
```

### 6. ⚙️ ADMIN DASHBOARD
```
├── Users (RBAC)
│   ├── User List
│   ├── Role Assignment (Operator/Analyst/DevOps/Infra/MLOps/Admin)
│   ├── Permission Matrix
│   ├── Invite/Deactivate User
│   └── Session Management
├── Feature Flags (LaunchDarkly)
│   ├── Flag List
│   ├── Enable/Disable Toggle
│   ├── Targeting Rules
│   └── Rollout Percentage
├── Audit Logs (YENİ)
│   ├── Who did What When
│   ├── Filter by User/Action/Resource
│   ├── Export Audit Trail
│   └── Compliance Reports
├── Secrets (Vault)
│   ├── Secret Paths Browser
│   ├── Masked Value View
│   ├── Rotation Status
│   └── Access History
└── System Config
    ├── Global Settings
    ├── API Keys Management
    ├── Rate Limit Config
    └── Broadcast Platform Config (Twitter/Telegram/Android)
```

---

## 📐 LAYOUT TASARIM GEREKSİNİMLERİ

### Header (Sabit, 56-64px)
```
┌─────────────────────────────────────────────────────────────────────────┐
│ [Logo] [Rol Tabs: Operator|Analyst|DevOps|Infra|MLOps|Admin]           │
│ [Global Search 🔍] [WS Status ●] [Kafka Lag] [Alerts 🔔] [User 👤]      │
└─────────────────────────────────────────────────────────────────────────┘
```

### Sidebar (Sol, 260-280px, Collapsible)
- **Dynamic:** Seçili role göre menü değişir
- **Primary Nav:** Modül listesi
- **Secondary Nav:** Alt menüler (contextual)
- **Icons:** Lucide veya Heroicons
- **States:** Aktif, Hover, Disabled

### Middle Area (Viewport)
- 12-column grid
- Card-based widgets
- Real-time data with RxJS streams
- WebGL for heavy visualizations (heatmaps)

### Footer (Non-sticky)
- Build/Commit info
- Model version (v3.1)
- Canary percentage
- Environment label

---

## 🎯 BEKLENEN ÇIKTILAR

Her katılımcı şu konularda detaylı tasarım sunmalı:

### 1. ROL BAZLI SAYFA HİYERARŞİSİ
- Her rol için sayfa/modül listesi
- Alt menü yapısı (tüm detaylar)
- Route yapısı (/operator/signal, /devops/logs, etc.)

### 2. LAYOUT VE NAVİGASYON
- Header içeriği ve davranışı
- Sidebar yapısı (rol bazlı dinamik)
- Rol geçiş mekanizması
- Breadcrumb yapısı

### 3. WİDGET VE BİLEŞEN TASARIMI
- Her modül için widget listesi
- Widget yerleşimi (grid coordinates)
- Real-time vs polling stratejisi
- Chart türleri (gauge, donut, line, heatmap, etc.)

### 4. KRİTİK YENİ MODÜLLER
- **Logs UI:** ELK/Loki entegrasyonu, search, filter, stream
- **Terminal/SSH:** xterm.js, pod exec, multi-tab
- **ML Platform:** MLflow, Ray, Feast, GE, Optuna UI tasarımı
- **Infrastructure:** K8s dashboard, ArgoCD controls

### 5. RBAC VE GÜVENLİK
- Hangi rol neyi görebilir?
- Permission matrix
- Audit logging UI

### 6. STATE VE DATA FLOW
- Global state yapısı (RxJS streams)
- Backend API entegrasyonu
- WebSocket channels

---

## 🔥 KRİTİK SORULAR

Panel şu soruları cevaplamalıdır:

1. **Rol geçişi nasıl olacak?** Header'da tab mı, dropdown mı?
2. **Terminal/SSH güvenliği nasıl sağlanacak?** Audit, timeout, permission
3. **Log streaming performansı?** Backpressure, virtualization
4. **ML Platform embed mi, link mi?** MLflow/Grafana embed edilecek mi?
5. **Audit log retention?** Ne kadar süre tutulacak?
6. **Real-time vs On-demand?** Hangi veriler canlı, hangileri istek üzerine?

---

## 📊 REFERANS SİSTEMLER

- **Grafana:** Monitoring dashboards, log explorer
- **Kubernetes Dashboard:** Pod/deployment management
- **MLflow UI:** Experiment tracking
- **ArgoCD UI:** GitOps deployments
- **Datadog:** Full-stack observability
- **Lens (K8s IDE):** Cluster management
- **Portainer:** Container management
- **Vault UI:** Secret management

---

## ⚠️ KISITLAR

1. **Masaüstü Odaklı:** Responsive tasarıma odaklanmayın, desktop-first
2. **Performans:** 60fps, TTI < 3s
3. **Dark Mode Öncelikli:** Deep Space tema
4. **WCAG 2.1 AA:** Accessibility
5. **Enterprise-Grade:** Production-ready, premium tasarım

---

## 📋 ÇIKTI FORMATI

```
[ROL: OPERATOR]
├── Modüller ve Alt Menüler
├── Widget Listesi ve Yerleşim
└── Routes

[ROL: ANALYST]
...

[ROL: DEVOPS]
...

[ROL: INFRASTRUCTURE]
...

[ROL: MLOPS]
...

[ROL: ADMIN]
...

[LAYOUT DETAYLARI]
├── Header
├── Sidebar (rol bazlı)
├── Middle Area
└── Footer

[KRİTİK MODÜL: LOGS]
├── UI Tasarımı
├── Search/Filter
└── Streaming

[KRİTİK MODÜL: TERMINAL/SSH]
├── UI Tasarımı
├── Security
└── Features

[KRİTİK MODÜL: ML PLATFORM]
...

[RBAC MATRİSİ]
├── Rol-Permission eşleşmesi

[SORU/ELEŞTİRİ]
└── Diğer katılımcılara sorular
```

---

Moderatör lütfen münazarayı başlatın. **Tüm sistemi kapsayan, rol bazlı, enterprise-grade UI tasarımı** için oy birliği hedefliyoruz.

---

**Referans Dokümanlar Ektedir:**
1. `bettingenesis-v3.1.md`
2. `BROADCAST_LAYER_v3.1.md`
3. `FRONTEND_ARCHITECTURE_v2.0.md`
