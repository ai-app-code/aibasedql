# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 6C
## Frontend - DevOps Modülleri (Logs ⭐⭐⭐) (4+ Seviye Derinlik)

**Tarih:** 05.01.2026 | **Referans:** FRONTEND_ARCHITECTURE_v2.1.md

---

```
superbet-genesis/frontend/modules/ (devam)
│
├── 📁 metrics/                    ← DevOps Dashboard
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       ├── @superbet/shared: workspace:*
│   │       └── recharts: ^2.10.0
│   │
│   ├── 📄 webpack.config.js
│   │   └── exposes: { './RemoteEntry': './src/index.tsx' }
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 PrometheusPanel.tsx
│   │   │   │   └── export const PrometheusPanel: React.FC = () => {
│   │   │   │       ├── const metrics = useSWR('/api/prometheus/query', fetchPrometheusMetrics)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Prometheus Metrics">
│   │   │   │           │   ├── <div className="metrics-grid">
│   │   │   │           │   │   ├── <MetricCard label="p99 Latency" value={metrics.p99_latency} unit="ms" />
│   │   │   │           │   │   ├── <MetricCard label="Request Rate" value={metrics.req_rate} unit="/s" />
│   │   │   │           │   │   ├── <MetricCard label="Error Rate" value={metrics.error_rate} unit="%" />
│   │   │   │           │   │   └── <MetricCard label="Kafka Lag" value={metrics.kafka_lag} unit="ms" />
│   │   │   │           │   └── </div>
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 BusinessKPIs.tsx
│   │   │       └── export const BusinessKPIs: React.FC = () => {
│   │   │           ├── const kpis = useSWR('/api/metrics/business', fetchBusinessKPIs)
│   │   │           └── return (
│   │   │               ├── <Card title="Business KPIs">
│   │   │               │   ├── <Gauge value={kpis.roi} min={-0.1} max={0.1} label="ROI" />
│   │   │               │   ├── <Gauge value={kpis.sharpe} min={0} max={3} label="Sharpe" />
│   │   │               │   └── <Gauge value={kpis.win_rate} min={0} max={1} label="Win Rate" />
│   │   │               └── </Card>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useMetrics.ts
│   │           └── export const useMetrics = () => useSWR('/api/metrics', fetchMetrics)
│   │
│   └── 📁 styles/
│       └── 📄 metrics.css
│
├── 📁 logs/                       ← DevOps (KRİTİK YENİ v2.1) ⭐⭐⭐
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       ├── @superbet/shared: workspace:*
│   │       ├── react-window: ^3.0.0
│   │       ├── rxjs: ^7.8.0
│   │       └── date-fns: ^2.30.0
│   │
│   ├── 📄 webpack.config.js
│   │   └── ModuleFederationPlugin:
│   │       ├── name: 'logs'
│   │       ├── exposes: { './RemoteEntry': './src/index.tsx' }
│   │       └── shared: ['react', 'react-window', 'rxjs']
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │   └── export { LogsModule as default } from './LogsModule'
│   │   │
│   │   ├── 📄 LogsModule.tsx
│   │   │   └── export const LogsModule: React.FC = () => {
│   │   │       ├── return (
│   │   │       │   ├── <div className="logs-module">
│   │   │       │   │   ├── <LogsHeader>
│   │   │       │   │   │   ├── <SearchPanel />
│   │   │       │   │   │   └── <StatsBar />
│   │   │       │   │   ├── </LogsHeader>
│   │   │       │   │   └── <LogStream />
│   │   │       │   └── </div>
│   │   │       └── )
│   │   │       }
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 LogStream.tsx
│   │   │   │   ├── import { FixedSizeList as List } from 'react-window'
│   │   │   │   ├── import AutoSizer from 'react-virtualized-auto-sizer'
│   │   │   │   ├── import { useLogs } from '../hooks/useLogs'
│   │   │   │   ├── import { useLogBackfill } from '../hooks/useLogBackfill'
│   │   │   │   └── export const LogStream: React.FC = () => {
│   │   │   │       ├── const logs = useLogs()
│   │   │   │       ├── const { backfill, isLoading } = useLogBackfill()
│   │   │   │       ├── const listRef = useRef<FixedSizeList>(null)
│   │   │   │       ├── // Cap at 10k lines
│   │   │   │       ├── const cappedLogs = useMemo(() => logs.slice(-10000), [logs])
│   │   │   │       ├── const LogRow = ({ index, style }) => {
│   │   │   │       │   ├── const log = cappedLogs[index]
│   │   │   │       │   └── return (
│   │   │   │       │       ├── <div style={style} className={`log-row log-${log.level.toLowerCase()}`}>
│   │   │   │       │       │   ├── <span className="log-timestamp">{formatTimestamp(log.timestamp)}</span>
│   │   │   │       │       │   ├── <span className="log-level">{log.level}</span>
│   │   │   │       │       │   ├── <span className="log-service">{log.service}</span>
│   │   │   │       │       │   └── <span className="log-message">{log.message}</span>
│   │   │   │       │       └── </div>
│   │   │   │       │   )
│   │   │   │       │   }
│   │   │   │       ├── useEffect(() => {
│   │   │   │       │   ├── // Auto-scroll to bottom on new logs
│   │   │   │       │   └── listRef.current?.scrollToItem(cappedLogs.length - 1)
│   │   │   │       │   }, [cappedLogs.length])
│   │   │   │       └── return (
│   │   │   │           ├── <div className="log-stream">
│   │   │   │           │   ├── {isLoading && <div className="loading-spinner">Loading older logs...</div>}
│   │   │   │           │   └── <AutoSizer>
│   │   │   │           │       {({ height, width }) => (
│   │   │   │           │         <List
│   │   │   │           │           ref={listRef}
│   │   │   │           │           height={height}
│   │   │   │           │           width={width}
│   │   │   │           │           itemCount={cappedLogs.length}
│   │   │   │           │           itemSize={24}  // 24px per log line
│   │   │   │           │           overscanCount={20}
│   │   │   │           │           onScroll={({ scrollOffset }) => {
│   │   │   │           │             // Trigger backfill when scrolling to top
│   │   │   │           │             if (scrollOffset < 100) {
│   │   │   │           │               backfill()
│   │   │   │           │             }
│   │   │   │           │           }}
│   │   │   │           │         >
│   │   │   │           │           {LogRow}
│   │   │   │           │         </List>
│   │   │   │           │       )}
│   │   │   │           │   </AutoSizer>
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 SearchPanel.tsx
│   │   │   │   └── export const SearchPanel: React.FC = () => {
│   │   │   │       ├── const [service, setService] = useState<string>('all')
│   │   │   │       ├── const [level, setLevel] = useState<string>('all')
│   │   │   │       ├── const [timeRange, setTimeRange] = useState<TimeRange>({ start: Date.now() - 3600000, end: Date.now() })
│   │   │   │       └── return (
│   │   │   │           ├── <div className="search-panel">
│   │   │   │           │   ├── <select value={service} onChange={e => setService(e.target.value)}>
│   │   │   │           │   │   ├── <option value="all">All Services</option>
│   │   │   │           │   │   ├── <option value="data-plant">Data Plant</option>
│   │   │   │           │   │   ├── <option value="intelligence">Intelligence</option>
│   │   │   │           │   │   └── <option value="stream-processor">Stream Processor</option>
│   │   │   │           │   ├── </select>
│   │   │   │           │   ├── <div className="level-chips">
│   │   │   │           │   │   ├── {['DEBUG', 'INFO', 'WARN', 'ERROR', 'FATAL'].map(l => (
│   │   │   │           │   │   │   <button
│   │   │   │           │   │   │     key={l}
│   │   │   │           │   │   │     className={`chip ${level === l ? 'active' : ''}`}
│   │   │   │           │   │   │     onClick={() => setLevel(l)}
│   │   │   │           │   │   │   >
│   │   │   │           │   │   │     {l}
│   │   │   │           │   │   │   </button>
│   │   │   │           │   │   └── ))}
│   │   │   │           │   ├── </div>
│   │   │   │           │   └── <TimeRangePicker value={timeRange} onChange={setTimeRange} />
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 StatsBar.tsx
│   │   │       └── export const StatsBar: React.FC = () => {
│   │   │           ├── const stats = useStream(logStats$, { rate: 0, total: 0, lag: 0, buffer: 0 })
│   │   │           └── return (
│   │   │               ├── <div className="stats-bar">
│   │   │               │   ├── <span className="stat">Rate: {stats.rate.toFixed(0)} logs/s</span>
│   │   │               │   ├── <span className="stat">Total: {stats.total.toLocaleString()}</span>
│   │   │               │   ├── <span className="stat">Lag: {stats.lag}ms</span>
│   │   │               │   └── <span className="stat">Buffer: {stats.buffer}</span>
│   │   │               └── </div>
│   │   │           )
│   │   │           }
│   │   │
│   │   ├── 📁 hooks/
│   │   │   ├── 📄 useLogs.ts
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   ├── import { logs$ } from '../streams/logs'
│   │   │   │   └── export const useLogs = () => {
│   │   │   │       ├── const logs = useStream(logs$, [])
│   │   │   │       └── return logs
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 useLogBackfill.ts
│   │   │       ├── import { useState, useCallback } from 'react'
│   │   │       └── export const useLogBackfill = () => {
│   │   │           ├── const [isLoading, setIsLoading] = useState(false)
│   │   │           ├── const [lastOffset, setLastOffset] = useState<number>(0)
│   │   │           ├── const backfill = useCallback(async () => {
│   │   │           │   ├── setIsLoading(true)
│   │   │           │   ├── try {
│   │   │           │   │   ├── const response = await fetch(`/api/logs/backfill?since=${lastOffset}&limit=500`)
│   │   │           │   │   ├── const logs = await response.json()
│   │   │           │   │   ├── if (logs.length > 0) {
│   │   │           │   │   │   ├── // Append to log stream
│   │   │           │   │   │   ├── logs$.next([...logs, ...currentLogs])
│   │   │           │   │   │   └── setLastOffset(logs[0].offset)
│   │   │           │   │   │   }
│   │   │           │   │   } finally {
│   │   │           │   │       └── setIsLoading(false)
│   │   │           │   │   }
│   │   │           │   }, [lastOffset])
│   │   │           └── return { backfill, isLoading }
│   │   │           }
│   │   │
│   │   ├── 📁 streams/
│   │   │   └── 📄 logs.ts
│   │   │       ├── import { Subject, BehaviorSubject } from 'rxjs'
│   │   │       ├── import { auditTime, windowCount, bufferTime, sampleTime, switchMap, map, observeOn, animationFrameScheduler } from 'rxjs/operators'
│   │   │       ├── // EWMA Rate Calculator
│   │   │       ├── class EWMARate {
│   │   │       │   ├── private alpha = 0.2  // Smoothing factor
│   │   │       │   ├── private rate = 0
│   │   │       │   ├── private lastTime = Date.now()
│   │   │       │   ├── update(count: number) {
│   │   │       │   │   ├── const now = Date.now()
│   │   │       │   │   ├── const delta = (now - this.lastTime) / 1000  // seconds
│   │   │       │   │   ├── const instantRate = count / delta
│   │   │   │   │   │   ├── this.rate = this.alpha * instantRate + (1 - this.alpha) * this.rate
│   │   │       │   │   ├── this.lastTime = now
│   │   │       │   │   └── return this.rate
│   │   │       │   │   }
│   │   │       │   └── getRate() { return this.rate }
│   │   │       │   }
│   │   │       ├── const rateCalculator = new EWMARate()
│   │   │       ├── // Raw log stream from WebSocket
│   │   │       ├── const rawLogs$ = new Subject<LogEntry>()
│   │   │       ├── // Hybrid EWMA backpressure pattern
│   │   │       ├── export const logs$ = rawLogs$.pipe(
│   │   │       │   ├── switchMap(log => {
│   │   │       │   │   ├── const rate = rateCalculator.getRate()
│   │   │       │   │   ├── // Low rate: auditTime(100) + windowCount(500)
│   │   │       │   │   ├── if (rate < 1000) {
│   │   │       │   │   │   └── return rawLogs$.pipe(
│   │   │       │   │   │       ├── auditTime(100),            // Throttle to 10Hz
│   │   │       │   │   │       └── windowCount(500)           // Batch max 500 logs
│   │   │       │   │   │       )
│   │   │       │   │   │   }
│   │   │       │   │   ├── // High rate (≥1000/s): bufferTime + sampleTime + windowCount
│   │   │       │   │   └── return rawLogs$.pipe(
│   │   │       │   │       ├── bufferTime(250),             // Buffer 250ms
│   │   │       │   │       ├── sampleTime(200),             // Sample at 5Hz
│   │   │       │   │       └── windowCount(500)             // Cap at 500
│   │   │       │   │       )
│   │   │       │   │   }),
│   │   │       │   ├── // Render on animation frames
│   │   │       │   ├── observeOn(animationFrameScheduler),
│   │   │       │   └── map(batch => batch.flat())
│   │   │       │   )
│   │   │       ├── // Stats stream
│   │   │       ├── export const logStats$ = new BehaviorSubject({ rate: 0, total: 0, lag: 0, buffer: 0 })
│   │   │       ├── // Update stats on each batch
│   │   │       ├── logs$.subscribe(batch => {
│   │   │       │   ├── const rate = rateCalculator.getRate()
│   │   │       │   ├── const total = logStats$.value.total + batch.length
│   │   │       │   └── logStats$.next({ rate, total, lag: 0, buffer: batch.length })
│   │   │       │   })
│   │   │       └── // WebSocket connection
│   │   │           const ws = new WebSocket('wss://bff.superbet:8080/ws/logs')
│   │   │           ws.onmessage = (event) => {
│   │   │             const log = JSON.parse(event.data)
│   │   │             rawLogs$.next(log)
│   │   │           }
│   │   │
│   │   └── 📁 styles/
│   │       └── 📄 logs.css
│   │           ├── .log-stream { height: calc(100vh - 200px); background: #0A0A0F; }
│   │           ├── .log-row { font-family: 'JetBrains Mono', monospace; font-size: 12px; }
│   │           ├── .log-debug { color: #6B7280; }
│   │           ├── .log-info { color: #FFFFFF; }
│   │           ├── .log-warn { color: #F59E0B; }
│   │           ├── .log-error { color: #EF4444; }
│   │           ├── .log-fatal { color: #DC2626; font-weight: bold; }
│   │           ├── .stats-bar { display: flex; gap: 16px; padding: 8px; background: #12121A; }
│   │           └── .level-chips .chip { padding: 4px 12px; border-radius: 16px; cursor: pointer; }
│   │               .level-chips .chip.active { background: var(--accent-primary); }
│   │
│   └── 📄 README.md
│       ├── # Logs Module
│       ├── ## Features
│       ├── - Real-time log streaming with EWMA fallback (Hybrid)
│       ├── - Virtualized rendering (react-window) with 10k line cap
│       ├── - Backfill REST endpoint for historical logs
│       ├── - Service/Level filtering
│       └── ## Performance
│           - Rate < 1000/s: auditTime(100) + windowCount(500)
│           - Rate ≥ 1000/s: bufferTime(250) + sampleTime(200) + windowCount(500)
│
├── 📁 traces/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 JaegerEmbed.tsx
│   │       │   └── export const JaegerEmbed: React.FC = () => {
│   │       │       └── return <iframe src="https://jaeger.superbet/traces" width="100%" height="600px" />
│   │       │       }
│   │       │
│   │       └── 📄 SpanVisualization.tsx
│   │           └── // D3.js span timeline
│   │
│   └── 📁 styles/
│       └── 📄 traces.css
│
├── 📁 alerts/                     ← DevOps (YENİ v2.1) ⭐
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 ActiveAlerts.tsx
│   │   │   │   └── export const ActiveAlerts: React.FC = () => {
│   │   │   │       ├── const alerts = useSWR('/api/alerts/active', fetchActiveAlerts)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Active Alerts">
│   │   │   │           │   └── <VirtualTable
│   │   │   │           │       data={alerts}
│   │   │   │           │       columns={[
│   │   │   │           │         { key: 'name', label: 'Alert' },
│   │   │   │           │         { key: 'severity', label: 'Severity' },
│   │   │   │           │         { key: 'timestamp', label: 'Time', format: formatTimestamp }
│   │   │   │           │       ]}
│   │   │   │           │     />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 AlertHistory.tsx
│   │   │   │   └── // Historical alerts with pagination
│   │   │   │
│   │   │   └── 📄 AcknowledgeButton.tsx
│   │   │       └── export const AcknowledgeButton: React.FC<{ alertId: string }> = ({ alertId }) => {
│   │   │           ├── const ack = async () => {
│   │   │           │   └── await fetch(`/api/alerts/${alertId}/ack`, { method: 'POST' })
│   │   │           │   }
│   │   │           └── return <button onClick={ack}>Acknowledge</button>
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useAlerts.ts
│   │           └── export const useAlerts = () => useSWR('/api/alerts', fetchAlerts)
│   │
│   └── 📁 styles/
│       └── 📄 alerts.css
│
└── 📁 health/
    ├── 📄 package.json
    ├── 📄 webpack.config.js
    │
    ├── 📁 src/
    │   ├── 📄 index.tsx
    │   │
    │   └── 📁 components/
    │       ├── 📄 ServiceStatus.tsx
    │       │   └── export const ServiceStatus: React.FC = () => {
    │       │       ├── const services = useSWR('/api/health/services', fetchServiceStatus)
    │       │       └── return (
    │       │           ├── <Card title="Service Health">
    │       │           │   └── <table className="health-table">
    │       │           │       {services.map(s => (
    │       │           │         <tr key={s.name}>
    │       │           │           <td>{s.name}</td>
    │       │           │           <td className={`status-${s.status}`}>{s.status}</td>
    │       │           │           <td>{s.uptime}h</td>
    │       │           │         </tr>
    │       │           │       ))}
    │       │           │     </table>
    │       │           └── </Card>
    │       │       )
    │       │       }
    │       │
    │       └── 📄 DependencyGraph.tsx
    │           ├── import * as d3 from 'd3'
    │           └── export const DependencyGraph: React.FC = () => {
    │               ├── // D3.js force-directed graph showing service dependencies
    │               └── return <svg ref={svgRef} width={800} height={600} />
    │               }
    │
    └── 📁 styles/
        └── 📄 health.css
            └── .health-table .status-healthy { color: var(--accent-success); }
                .health-table .status-degraded { color: var(--accent-warning); }
                .health-table .status-down { color: var(--accent-danger); }
```

---

## 📊 BÖLÜM C İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Modüller** | 5 (metrics, logs, traces, alerts, health) |
| **Toplam Dosya** | 25+ |
| **React Bileşenleri** | 15+ |
| **RxJS Streams** | 2 (logs$, logStats$) |
| **KRİTİK** | Logs ⭐⭐⭐ (EWMA fallback, react-window, backfill) |

---

## 🔗 DEVAMI

**Sonraki Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_D.md`  
**İçerik:** Infrastructure Modülleri (K8s, ArgoCD, **Terminal ⭐⭐⭐**, Config)

---

**Versiyon:** SUPERBET GENESIS v3.1 Frontend Tree - PART 6C  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md
