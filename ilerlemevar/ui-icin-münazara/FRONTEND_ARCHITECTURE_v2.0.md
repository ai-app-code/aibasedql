# 🎨 SUPERBET GENESIS v3.1 - FRONTEND MİMARİ PLANI
## Süper-Rasyonel Dijital Bahis Varlığı - Enterprise UI Blueprint

**Oluşturma:** 04.01.2026  
**Kaynak:** TETRA AI Panel Frontend Münazarası (10 LLM, 3 Tur)  
**Versiyon:** v2.0 (Modüler SPA + Go BFF + RxJS/Protobuf + Broadcast Layer Entegrasyonu)  
**Panel Katılımcıları:** Gemini 2.5/3, GPT-5, GPT-4o-mini, Grok 4.1, Kimi K2, Qwen3, GLM 4.7, MiniMax M2.1

> ⚡ **KONSEPT:** Signal-First Command Center + Broadcast Layer görünürlüğü. Go BFF ile mTLS proxy, RxJS stream tabanlı real-time, WebGL ile GNN simülasyonları.

> ⚠️ **v2.0 CHANGELOG:** Monolitik SPA → Modüler SPA, Zustand → RxJS, JSON → Protobuf, Dashboard-first → Signal-first, Broadcast Layer metrikleri eklendi.

---

# 📌 BÖLÜM 0: VİZYON VE FELSEFE

## Kritik Tasarım Kararları

| Soru | v1.0 Karar | v2.0 Karar | Gerekçe |
|------|------------|------------|---------|
| **Mimari?** | Monolitik SPA | **Modüler SPA** | Webpack MF ile domain-based modüller |
| **BFF var mı?** | Yok | **Go BFF zorunlu** | mTLS/Vault, Kafka proxy |
| **State?** | Zustand | **RxJS Streams** | 1M/s Kafka akışı için |
| **Veri formatı?** | JSON | **Protobuf (Binary)** | %60 payload azalma |
| **UX felsefesi?** | Dashboard-first | **Signal-first** | VSNR/CAS alarm odaklı |
| **Render?** | DOM only | **DOM + WebGL** | GNN simülasyonu için |
| **Worker?** | Yok | **SharedWorker** | Sekmeler arası tek WS |

## SLO Hedefleri

| Metrik | Hedef | Alert Threshold |
|--------|-------|-----------------|
| **TTI (Time to Interactive)** | < 3s | > 5s |
| **p99 WS Latency** | < 60ms | > 80ms |
| **FPS** | > 60fps | < 50fps |
| **JS Bundle (gzip)** | < 250KB | > 400KB |
| **WCAG Uyumluluk** | 2.1 AA | - |

---

# 🏗️ SAYFA MİMARİSİ

## Modüler SPA Yapısı

```
📱 SUPERBET GENESIS UI (Modüler SPA)
│
├── 🐚 Shell App (Ana Container)
│   ├── Layout (Header, Sidebar, Footer)
│   ├── Auth/mTLS Proxy
│   ├── Global WS Connection (SharedWorker)
│   ├── Feature Flags (LaunchDarkly)
│   ├── Theme/Density Provider
│   └── Error Boundaries
│
├── ⚡ Signal Center (Ana Sayfa - Signal-First)
│   ├── VSNR/CAS Alarm Feed (WS Stream)
│   ├── EDL Uncertainty Heatmap (WebGL)
│   ├── Risk Limit Status (Bars)
│   ├── ROI/Sharpe KPI Cards
│   ├── ✨ Broadcast Live Ticker
│   └── γ Gamma Rejim Göstergesi
│
├── ⚽ Matches Module (L1)
│   ├── Match List (Pre/Live)
│   ├── Match Detail (Timeline, xG, Odds)
│   └── Pre→Live Handover Durumu
│
├── 🎯 Predictions Module (L1)
│   ├── HRL Önerileri
│   ├── Kupon Kombinasyonları
│   ├── Kelly Pay Dağılımı
│   └── ✨ Broadcast Yayın Durumu
│
├── ⚠️ Risk Module (L1)
│   ├── VaR/CVaR/MaxDrawdown/Sharpe
│   ├── Limit Enforcement
│   └── CB Status Matrix
│
├── 📊 Strategies Module (L2)
│   ├── UCB Manager ROI
│   ├── BCD Rejim Algılama
│   └── Strateji Ağırlıkları
│
├── 📈 Monitoring Module (L2)
│   ├── EDL Kalibrasyon (PIT/ECE)
│   ├── Circuit Breaker Matrix
│   ├── ✨ Broadcast Metrics (Platform CB/Rate Limit)
│   └── WS/Render Latency
│
└── ⚙️ Settings Module (L2)
    ├── Tema/Density
    ├── ✨ Broadcast Platform Tercihleri
    └── Feature Flags
```

## Routing Stratejisi

| Route | Modül | Lazy Load | Prefetch |
|-------|-------|-----------|----------|
| `/` | Signal Center | ❌ (Kritik) | - |
| `/matches` | Matches List (Sanal Scroll) | ✅ | viewport yakınlığı |
| `/matches/:id` | Match Detail | ✅ | hover ile |
| `/predictions` | Predictions | ✅ | viewport yakınlığı |
| `/predictions/:id` | Coupon Detail + EDL Explainability | ✅ | hover ile |
| `/risk` | Risk Dashboard | ✅ | idle zamanı |
| `/strategies` | Strategy Overview | ✅ | idle zamanı |
| `/monitoring` | Monitoring | ✅ | idle zamanı |
| `/settings` | Settings | ✅ | idle zamanı |

> 📌 **Prefetch Stratejisi:** `viewport'a yakın modül için preload hint` + `IntersectionObserver` ile dinamik yükleme

```tsx
// shell/src/App.tsx (Modüler SPA)
import { loadRemote } from '@module-federation/webpack-remotes';

const SignalCenter = React.lazy(() => loadRemote('./signal-center/RemoteEntry'));
const MatchesModule = React.lazy(() => loadRemote('./matches/RemoteEntry'));
const RiskModule = React.lazy(() => loadRemote('./risk/RemoteEntry'));

export const App = () => (
  <ShellLayout>
    <Suspense fallback={<SkeletonLoader />}>
      <Routes>
        <Route path="/" element={<SignalCenter />} />
        <Route path="/matches/*" element={<MatchesModule />} />
        <Route path="/risk/*" element={<RiskModule />} />
        {/* ... */}
      </Routes>
    </Suspense>
  </ShellLayout>
);
```

---

# 📐 LAYOUT SİSTEMİ

## Genel Layout Blueprint

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  HEADER (56-64px, fixed)                                                    │
│  [Logo] [Global Search] [WS Status] [Kafka Lag] [SLO p99] [🔔] [👤]         │
├───────────┬─────────────────────────────────────────────────────────────────┤
│           │                                                                 │
│  SIDEBAR  │                    MIDDLE AREA                                  │
│ (260-280px│                    (12-col Responsive Grid)                     │
│ collapsible│                                                                │
│           │  ┌────────────────────────────────────────────────────────┐    │
│  [⚡ Signal│  │                                                        │    │
│  [⚽ Matches│ │          SIGNAL CENTER / MODULE CONTENT               │    │
│  [🎯 Preds]│  │                                                        │    │
│  [⚠️ Risk] │  │          (Widget Grid with RxJS Streams)              │    │
│  [📊 Strategy│ │                                                        │    │
│  [📈 Monitor]│ └────────────────────────────────────────────────────────┘    │
│  [⚙️ Settings]│                                                             │
│           │                                                                 │
├───────────┴─────────────────────────────────────────────────────────────────┤
│  FOOTER (non-sticky)                                                        │
│  [Build: abc123] [Model: v3.1] [Canary: 10%] [Env: production]             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Header Spesifikasyonu

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Yükseklik** | 56-64px | Responsive |
| **Pozisyon** | Fixed | Sticky top |
| **Sol İçerik** | Logo + Global Search | Fuzzy arama (lig/takım/maç) |
| **Orta İçerik** | WS Status, Kafka Lag, **Freshness > 0.3 SLO**, p99 latency, skip rate | Real-time indicators |
| **Sağ İçerik** | Hedge Now, New Simulation, User Menu, Theme Toggle | Quick actions |
| **Kısayol** | `K` veya `/` | Arama açar |

## Sidebar Spesifikasyonu

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Genişlik** | 260-280px (collapsed: 64px) | Mobile: hamburger |
| **Pozisyon** | Fixed left | Dock |
| **Primary Nav** | Signal, Matches, Predictions, Risk, Strategies, Monitoring, Settings | |
| **Secondary Nav** | Contextual alt menüler | Seçili modüle göre |
| **Mobile** | Overlay + hamburger | Slide-in |
| **4K** | Geniş ikon + metin | Tooltip |

## Middle Area

| Özellik | Değer | Detay |
|---------|-------|-------|
| **Grid Sistemi** | 12-column | TailwindCSS |
| **Breakpoints** | sm(640), md(768), lg(1024), xl(1280), 2xl(1536), **4K(1920+)** | Mobile-first |
| **Card Yapısı** | shadow-md, rounded-lg | Virtualized tables |

---

# 📊 SIGNAL CENTER WIDGET'LARI (Signal-First)

## Ana Sayfa Yerleşimi

```
┌──────────────────────────────────────────────────────────────────┐
│  SIGNAL CENTER (Ana Sayfa, Signal-First)                         │
├────────────────────────┬─────────────────────────────────────────┤
│  VSNR/CAS Alarm Feed   │  EDL Uncertainty Heatmap (WebGL)        │
│  x:0 y:0 w:8 h:6       │  x:8 y:0 w:4 h:6                        │
│  [RxJS Stream → DOM]   │  [α-sum/entropy, lig x saat]            │
├────────────────────────┼─────────────────────────────────────────┤
│  Risk Limit Status     │  ROI/Sharpe KPI Cards                   │
│  x:0 y:6 w:4 h:3       │  x:4 y:6 w:4 h:3                        │
│  [Günlük/Haftalık bar] │  [Prometheus gauge + sparkline]         │
├────────────────────────┴─────────────────────────────────────────┤
│  ✨ BROADCAST LIVE TICKER (Full Width)                           │
│  x:0 y:9 w:12 h:2                                                │
│  [risk.verified → broadcast.queue.priority → yayınlanan]         │
│  [Filtre highlight: Conf>0.65, VSNR>2.2, CAS>1.0]                │
├─────────────────────────────────────────────────────────────────┤
│  γ Gamma Rejim Göstergesi │ CB Status Donut                       │
│  x:0 y:11 w:6 h:2        │ x:6 y:11 w:6 h:2                       │
│  [Liderlik/Eşgüdüm]      │ [OPEN/CLOSED/HALF_OPEN dağılımı]       │
└─────────────────────────────────────────────────────────────────┘
```

## Widget Detayları

| Widget | Konum | Veri Kaynağı | Render | Real-time |
|--------|-------|--------------|--------|-----------|
| **VSNR/CAS Alarm Feed** | 8×6 | Kafka WS (Protobuf) | RxJS → DOM List | ✅ WS |
| **EDL Uncertainty Heatmap** | 4×6 | EDL Metrics | WebGL (Canvas) | ✅ WS |
| **Risk Limit Status** | 4×3 | Redis/VaR/CVaR | DOM Bars | ✅ WS |
| **ROI/Sharpe KPI** | 4×3 | Prometheus | Gauge + Sparkline | ✅ 5s poll |
| **✨ Broadcast Ticker** | 12×2 | broadcast.queue.priority | DOM List | ✅ WS |
| **γ Gamma Rejim** | 6×2 | HRL Agents | Threshold Indicator | ✅ WS |
| **CB Status Donut** | 6×2 | Circuit Breakers | Donut Chart | ✅ WS |

---

# ✨ BROADCAST LAYER ENTEGRASİYONU

## Broadcast Widget'ları

### 1. Broadcast Live Ticker

```tsx
// components/BroadcastTicker.tsx
import { useSyncExternalStore } from 'react';
import { broadcastStream$ } from '../streams/broadcast';

export const BroadcastTicker = () => {
  const broadcasts = useSyncExternalStore(
    broadcastStream$.subscribe,
    () => broadcastStore.value
  );
  
  return (
    <div className="broadcast-ticker">
      {broadcasts.map(event => (
        <BroadcastCard
          key={event.id}
          event={event}
          isFiltered={
            event.metrics.confidence > 0.65 &&
            event.metrics.vsnr > 2.2 &&
            event.metrics.cas > 1.0
          }
        />
      ))}
    </div>
  );
};
```

### 2. Broadcast Metrics Panel (Monitoring Modülü)

| Metrik | Gösterim | Kaynak |
|--------|----------|--------|
| **broadcast_published_total** | Counter (platform bazlı) | Prometheus |
| **broadcast_dropped_total** | Counter (reason bazlı) | Prometheus |
| **broadcast_circuit_breaker_state** | Platform CB Matrix (0/1/2) | Prometheus |
| **broadcast_digest_buffer_size** | Gauge (platform bazlı) | Prometheus |
| **broadcast_latency_seconds** | Histogram (p50/p95/p99) | Prometheus |

### 3. Platform Rate Limit Durumu (Settings)

```
┌─────────────────────────────────────────────────────────────────┐
│  BROADCAST PLATFORM DURUMU                                      │
├─────────────────────┬───────────────────┬───────────────────────┤
│  Twitter            │  Telegram         │  Android              │
│  ───────────────    │  ───────────────  │  ───────────────      │
│  Günlük: 42/50 ✅   │  Günlük: 180/200 ✅│  Günlük: 850/1000 ✅  │
│  Saatlik: 8/10 ⚠️   │  Saatlik: 25/30 ✅ │  Saatlik: 90/100 ⚠️   │
│  CB: CLOSED ✅      │  CB: CLOSED ✅     │  CB: HALF_OPEN ⚠️     │
│  Buffer: 0          │  Buffer: 0         │  Buffer: 5            │
└─────────────────────┴───────────────────┴───────────────────────┘
```

---

# ⚡ REAL-TIME MİMARİSİ

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
       │                    │ (prod)    │                    │
       │                    └─────┬─────┘                    │
       │                          │                          │
       │                    ┌─────┴─────┐                    │
       │                    │  RxJS     │                    │
       │                    │  Subject  │                    │
       │                    │  (backpressure)               │
       │                    └─────┬─────┘                    │
       │                          │                          │
       │                    ┌─────┴─────┐                    │
       │                    │bufferTime │                    │
       │                    │  250ms    │                    │
       │                    └─────┬─────┘                    │
       │                          │                          │
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

## Go BFF Konfigürasyonu

```go
// bff/main.go
package main

import (
    "github.com/gorilla/websocket"
    "github.com/segmentio/kafka-go"
)

var upgrader = websocket.Upgrader{
    CheckOrigin: mTLSCheck, // Vault/SPIFFE identity
}

func wsHandler(w http.ResponseWriter, r *http.Request) {
    conn, _ := upgrader.Upgrade(w, r, nil)
    defer conn.Close()
    
    reader := kafka.NewReader(kafka.ReaderConfig{
        Brokers: []string{"kafka:9092"},
        Topic:   "risk.verified",
        GroupID: "bff-frontend",
    })
    
    for {
        msg, _ := reader.ReadMessage(context.Background())
        // Binary forward - no JSON conversion
        conn.WriteMessage(websocket.BinaryMessage, msg.Value)
    }
}

func main() {
    http.HandleFunc("/ws/alarms", wsHandler)
    http.HandleFunc("/ws/matches", matchesHandler)
    http.HandleFunc("/ws/broadcast", broadcastHandler) // ✨ Yeni
    log.Fatal(http.ListenAndServeTLS(":8080", cert, key, nil))
}
```

## WS Kanalları

| Kanal | Topic | Payload | Header |
|-------|-------|---------|--------|
| `/ws/alarms` | risk.verified | CloudEvents (Protobuf) | **priority_score** |
| `/ws/matches` | prematch, live | TwinDelta (Protobuf) | match_id, phase |
| `/ws/broadcast` | broadcast.queue.priority | BroadcastData (Protobuf) | platform, cb_state |
| `/ws/metrics` | Prometheus scrape | JSON | - |

---

# 🔄 STATE YÖNETİMİ (RxJS)

## Stream Mimarisi

```typescript
// streams/index.ts
import { Subject, BehaviorSubject } from 'rxjs';
import { bufferTime, shareReplay, map } from 'rxjs/operators';
import { decode } from 'protobufjs';

// SharedWorker'dan gelen raw binary
const rawBinary$ = new Subject<ArrayBuffer>();

// Protobuf decode
const decoded$ = rawBinary$.pipe(
  map(buffer => decode(TwinDelta, new Uint8Array(buffer))),
  shareReplay({ bufferSize: 50, windowTime: 60000 })
);

// Domain streams with FULL backpressure strategy
export const alarms$ = decoded$.pipe(
  filter(msg => msg.type === 'alarm'),
  bufferTime(250),           // Batch 250ms
  sampleTime(200),           // UI-priority sampling
  auditTime(500),            // Low-priority throttle
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
```

## React Entegrasyonu

```typescript
// hooks/useStream.ts
import { useSyncExternalStore, useCallback } from 'react';
import { Observable } from 'rxjs';

export function useStream<T>(stream$: Observable<T>, initialValue: T): T {
  const subscribe = useCallback(
    (callback: () => void) => {
      const sub = stream$.subscribe(callback);
      return () => sub.unsubscribe();
    },
    [stream$]
  );
  
  const getSnapshot = useCallback(() => {
    // Store reference from stream
    return streamStore.get(stream$) ?? initialValue;
  }, [stream$, initialValue]);
  
  return useSyncExternalStore(subscribe, getSnapshot);
}

// Usage
const alarms = useStream(alarms$, []);
const broadcasts = useStream(broadcast$, []);
```

## SharedWorker

```typescript
// worker/shared.ts
const sockets = new Map<string, WebSocket>();

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
    
    sockets.set('main', ws);
  }
  
  port.start();
};

const channel = new BroadcastChannel('superbet-realtime');
```

---

# 🎨 TEMA SİSTEMİ

## Design Tokens

```css
:root {
  /* Background (Dark Mode Default) */
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
  
  /* Uncertainty Mapping (EDL τ) */
  --uncertainty-low: #10B981;    /* τ < 0.2, doygun yeşil */
  --uncertainty-medium: #F59E0B; /* 0.2 < τ < 0.4, sarı */
  --uncertainty-high: #6B7280;   /* τ > 0.4, desaturated/striped */
  
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
  --font-mono: 'JetBrains Mono', monospace;
  
  /* Font Sizes */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 24px;
  --text-2xl: 32px;
}

/* Light Mode Override */
@media (prefers-color-scheme: light) {
  :root {
    --bg-primary: #FFFFFF;
    --bg-secondary: #F9FAFB;
    --text-primary: #111827;
  }
}
```

## Density Toggle

```css
/* Compact Mode */
.density-compact {
  --table-row-height: 32px;
  --card-padding: 8px;
  --widget-gap: 8px;
}

/* Cozy Mode (Default) */
.density-cozy {
  --table-row-height: 48px;

/* τ > 0.4 Uncertainty Lock State */
.uncertainty-locked {
  opacity: 0.5;
  background: repeating-linear-gradient(
    45deg,
    var(--bg-tertiary),
    var(--bg-tertiary) 10px,
    var(--bg-elevated) 10px,
    var(--bg-elevated) 20px
  );
  pointer-events: none; /* Eylem kilidi */
}

/* CB OPEN Warning Banner */
.cb-open-banner {
  background: var(--accent-warning);
  color: var(--bg-primary);
  padding: var(--space-2) var(--space-4);
  font-weight: 600;
  position: sticky;
  top: 64px;
  z-index: 100;
}

/* Cozy Mode Continued */
.density-cozy {
  --card-padding: 16px;
  --widget-gap: 16px;
}
```

## Bileşen Kütüphanesi

| Bileşen | Amaç | Render |
|---------|------|--------|
| **Card** | Widget container | DOM/CSS |
| **Gauge** | KPI göstergeleri (ROI, Sharpe) | SVG |
| **Donut** | CB Status dağılımı | SVG |
| **Sparkline** | Trend mini grafikler | SVG |
| **Virtual Table** | Büyük listeler (Matches, Predictions) | DOM + windowing |
| **Heatmap** | EDL Uncertainty, VSNR/CAS | WebGL (Canvas) |
| **Timeline** | Match xG alan grafiği | SVG |
| **AlertBanner** | CB OPEN sarı şerit, Digest Buffer bildirimi | DOM |

---

# 💾 CACHE STRATEJİSİ

## Stale-While-Revalidate (SWR) Pattern

```typescript
// cache/swr.ts
import { BehaviorSubject, merge } from 'rxjs';
import { switchMap, tap, startWith } from 'rxjs/operators';

class SWRCache<T> {
  private cache$ = new BehaviorSubject<T | null>(null);
  private staleTime = 30000; // 30s
  private lastFetch = 0;
  
  get(key: string, fetcher: () => Promise<T>): Observable<T> {
    const cached = this.cache$.value;
    const isStale = Date.now() - this.lastFetch > this.staleTime;
    
    if (cached && !isStale) {
      return of(cached);
    }
    
    // Stale-while-revalidate: Önce stale veriyi göster, arka planda yenile
    const fresh$ = from(fetcher()).pipe(
      tap(data => {
        this.cache$.next(data);
        this.lastFetch = Date.now();
        // IndexedDB'ye persist
        idb.set(key, data);
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

// Kullanım
const matchCache = new SWRCache<Match>();
const match$ = matchCache.get(`match:${id}`, () => api.getMatch(id));
```

---

# 🔌 BFF ENTEGRASİYON ŞEMASI

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SİSTEM ENTEGRASYON HARİTASI                          │
└─────────────────────────────────────────────────────────────────────────────┘

  Browser (Modüler SPA)                     Go BFF (:8080)
  ─────────────────────                     ─────────────────
  │                                         │
  │ WebSocket (Binary)                      │ ← mTLS Auth
  ├────────────────────────────────────────►│ ← Vault Secrets
  │                                         │
  │                                         ├── Kafka Consumer
  │                                         │   ├── risk.verified
  │                                         │   ├── prematch, live
  │                                         │   └── broadcast.queue.priority
  │                                         │
  │                                         ├── Redis (Rate Limits)
  │                                         │   └── broadcast:limits:*
  │                                         │
  │                                         └── Prometheus Query
  │                                             └── /api/v1/query
  │
  │ REST API (JSON)
  ├────────────────────────────────────────►│
  │ GET /api/matches/:id                    │ ← Feast Features
  │ GET /api/predictions                    │ ← ClickHouse
  │ POST /api/hedge                         │ ← Risk Actions

  Shell App                          Remote Modules (Webpack MF)
  ─────────                          ──────────────────────────
  │                                  │
  │ loadRemote('./signal-center')    │
  ├─────────────────────────────────►│ signal-center (CDN)
  │                                  │
  │ loadRemote('./matches')          │
  ├─────────────────────────────────►│ matches (CDN)
  │                                  │
  │ loadRemote('./risk')             │
  ├─────────────────────────────────►│ risk (CDN)
```

---

# 🚀 DEPLOYMENT

## Helm Chart Yapısı

```yaml
# infra/helm/charts/frontend-shell/values.yaml
replicaCount: 2

image:
  repository: superbet/frontend-shell
  tag: v2.0

bff:
  enabled: true
  image: superbet/go-bff:v1.0
  port: 8080
  kafka:
    brokers: ["kafka:9092"]
    topics:
      - risk.verified
      - prematch
      - live
      - broadcast.queue.priority
  redis:
    host: redis-cluster:6379

modules:
  - name: signal-center
    url: https://cdn.superbet/signal-center/remoteEntry.js
  - name: matches
    url: https://cdn.superbet/matches/remoteEntry.js
  - name: risk
    url: https://cdn.superbet/risk/remoteEntry.js

canary:
  enabled: true
  weight: 10  # %10 trafik

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 256Mi
```

## CI/CD Pipeline

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
      
      - name: Build Shell
        run: |
          cd frontend/shell
          npm ci
          npm run build
          
      - name: Build Modules
        run: |
          for module in signal-center matches risk; do
            cd frontend/$module
            npm ci && npm run build
          done
          
      - name: Push to CDN
        run: aws s3 sync dist/ s3://superbet-cdn/frontend/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Update Helm
        run: |
          yq e '.frontend.image.tag = "${{ github.sha }}"' -i helm/values.yaml
          git commit -m "Deploy frontend ${{ github.sha }}"
          git push
```

---

# 📊 NİHAİ KARAR MATRİSİ

| Kategori | Karar | Teknoloji | Gerekçe |
|----------|-------|-----------|---------|
| **Mimari** | Modüler SPA | Webpack Module Federation | Runtime basitliği |
| **BFF** | Go BFF | Go + Gorilla WS | Concurrency, p99 +2-6ms |
| **State** | RxJS Streams | RxJS 7 | 1M/s backpressure |
| **Transport** | Protobuf | protobuf.js | %60 payload azalma |
| **UX** | Signal-First | VSNR/CAS odaklı | Operatör aksiyonu |
| **Render (KPI)** | DOM/SVG | React + Tailwind | Basitlik |
| **Render (GNN)** | WebGL | Deck.gl/PixiJS | Performans |
| **Worker** | SharedWorker | + BroadcastChannel | Tek WS, cross-tab |
| **Cache** | shareReplay + IDB | RxJS + IndexedDB | Offline support |

---

# 📋 NİHAİ İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Toplam Modül** | 7 (Shell + 6 domain) |
| **Widget Tipleri** | 20+ |
| **Bileşen Kütüphanesi** | 8 (Card, Gauge, Donut, Sparkline, Virtual Table, Heatmap, Timeline, AlertBanner) |
| **WS Kanalları** | 4 |
| **RxJS Streams** | 8+ (domain bazlı) |
| **SLO Metrikleri** | 5 |
| **Cache Stratejisi** | SWR + IndexedDB |
| **Panel Katılımcı** | 10 LLM |
| **Münazara Turu** | 3 |
| **Broadcast Entegrasyonu** | ✅ Tam |
| **HIGH ATTENTION Kontrol** | ✅ Yapıldı |

---

**Kaynak:** TETRA AI Panel Frontend Münazarası (Broadcast Layer dahil)  
**Versiyon:** v2.0 GENESIS UI  
**Tarih:** 04.01.2026  
**Referans:** bettingenesis-v3.1.md + BROADCAST_LAYER_v3.1.md  
**Misyon:** Signal-First Command Center + Broadcast Layer görünürlüğü
