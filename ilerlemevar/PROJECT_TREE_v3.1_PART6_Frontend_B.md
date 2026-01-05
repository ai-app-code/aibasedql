# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 6B
## Frontend - Operator + Analyst Modülleri (4+ Seviye Derinlik)

**Tarih:** 05.01.2026 | **Referans:** FRONTEND_ARCHITECTURE_v2.1.md

---

```
superbet-genesis/frontend/modules/ (devam)
│
├── 📁 signal-center/              ← Operator Default Page ⭐
│   ├── 📄 package.json
│   │   ├── name: "@superbet/signal-center"
│   │   └── dependencies:
│   │       ├── react: ^18.2.0
│   │       ├── @superbet/shared: workspace:*
│   │       └── deck.gl: ^8.9.0
│   │
│   ├── 📄 webpack.config.js
│   │   └── ModuleFederationPlugin:
│   │       ├── name: 'signalCenter'
│   │       ├── filename: 'remoteEntry.js'
│   │       ├── exposes: { './RemoteEntry': './src/index.tsx' }
│   │       └── shared: ['react', 'react-dom', '@superbet/shared']
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │   └── export { SignalCenter as default } from './SignalCenter'
│   │   │
│   │   ├── 📄 SignalCenter.tsx
│   │   │   └── export const SignalCenter: React.FC = () => {
│   │   │       ├── return (
│   │   │       │   └── <SignalCenterGrid>
│   │   │       │       ├── <VSNRCASFeed />
│   │   │       │       ├── <LiveMatchesTicker />
│   │   │       │       ├── <RiskLimitBars />
│   │   │       │       ├── <BroadcastLiveTicker />
│   │   │       │       ├── <GammaGauge />
│   │   │       │       └── <CircuitBreakerDonut />
│   │   │       │   └── </SignalCenterGrid>
│   │   │       └── )
│   │   │       }
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 VSNRCASFeed.tsx
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   ├── import { alarms$ } from '@superbet/shared/streams'
│   │   │   │   ├── import { VirtualTable } from '@superbet/shared/components'
│   │   │   │   └── export const VSNRCASFeed: React.FC = () => {
│   │   │   │       ├── const alarms = useStream(alarms$, [])
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="VSNR/CAS Alarm Feed">
│   │   │   │           │   └── <VirtualTable
│   │   │   │           │       data={alarms}
│   │   │   │           │       columns={[
│   │   │   │           │         { key: 'match_id', label: 'Match' },
│   │   │   │           │         { key: 'vsnr', label: 'VSNR', format: v => v.toFixed(2) },
│   │   │   │           │         { key: 'cas', label: 'CAS', format: v => v.toFixed(2) },
│   │   │   │           │         { key: 'confidence', label: 'Conf', format: v => `${(v*100).toFixed(0)}%` }
│   │   │   │           │       ]}
│   │   │   │           │     />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 EDLHeatmap.tsx
│   │   │   │   ├── import { Heatmap } from '@superbet/shared/components'
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   └── export const EDLHeatmap: React.FC = () => {
│   │   │   │       ├── const predictions = useStream(predictions$, [])
│   │   │   │       ├── const heatmapData = useMemo(() => 
│   │   │   │       │   predictions.map(p => ({
│   │   │   │       │     x: p.vsnr,
│   │   │   │       │     y: p.cas,
│   │   │   │       │     value: p.uncertainty  // EDL τ
│   │   │   │       │   })), [predictions])
│   │   │   │       └── return <Card title="EDL Uncertainty Heatmap">
│   │   │   │           <Heatmap data={heatmapData} />
│   │   │   │           </Card>
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 BroadcastTicker.tsx
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   ├── import { broadcast$ } from '@superbet/shared/streams'
│   │   │   │   └── export const BroadcastTicker: React.FC = () => {
│   │   │   │       ├── const broadcasts = useStream(broadcast$, [])
│   │   │   │       └── return (
│   │   │   │           ├── <div className="broadcast-ticker">
│   │   │   │           │   └── {broadcasts.map(b => (
│   │   │   │           │       <div key={b.id} className={`ticker-item ${b.sent ? 'sent' : 'pending'}`}>
│   │   │   │           │         <span className="platform">{b.platform}</span>
│   │   │   │           │         <span className="text">{b.formatted_text.slice(0, 50)}...</span>
│   │   │   │           │       </div>
│   │   │   │           │     ))}
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 GammaGauge.tsx
│   │   │   │   ├── import { Gauge } from '@superbet/shared/components'
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   ├── import { agents$ } from '@superbet/shared/streams'
│   │   │   │   └── export const GammaGauge: React.FC = () => {
│   │   │   │       ├── const agents = useStream(agents$, [])
│   │   │   │       ├── const gamma = agents[0]?.gamma ?? 0.5
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="γ Gamma Rejim">
│   │   │   │           │   ├── <Gauge value={gamma} min={0} max={1} label="Liderlik/Eşgüdüm" />
│   │   │   │           │   └── <p>{gamma > 0.7 ? 'Liderlik Modu' : gamma > 0.3 ? 'Eşgüdüm Modu' : 'Nötr'}</p>
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 CircuitBreakerDonut.tsx
│   │   │       ├── import { Donut } from '@superbet/shared/components'
│   │   │       ├── import { circuitBreakers$ } from '@superbet/shared/streams'
│   │   │       └── export const CircuitBreakerDonut: React.FC = () => {
│   │   │           ├── const cbs = useStream(circuitBreakers$, [])
│   │   │           ├── const data = useMemo(() => {
│   │   │           │   └── const counts = { CLOSED: 0, OPEN: 0, HALF_OPEN: 0 }
│   │   │           │       cbs.forEach(cb => counts[cb.state]++)
│   │   │           │       return Object.entries(counts).map(([state, count]) => ({ label: state, value: count }))
│   │   │           │   }, [cbs])
│   │   │           └── return <Card title="Circuit Breaker Status"><Donut data={data} /></Card>
│   │   │           }
│   │   │
│   │   └── 📁 widgets/
│   │       └── 📄 SignalCenterGrid.tsx
│   │           └── export const SignalCenterGrid: React.FC<PropsWithChildren> = ({ children }) => (
│   │               ├── <div className="grid-12">
│   │               │   └── {children}
│   │               └── </div>
│   │               )
│   │
│   └── 📁 styles/
│       └── 📄 signal-center.css
│           ├── .grid-12 { display: grid; grid-template-columns: repeat(12, 1fr); gap: 16px; }
│           └── .broadcast-ticker { overflow-x: auto; white-space: nowrap; }
│
├── 📁 matches/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │   └── exposes: { './RemoteEntry': './src/index.tsx' }
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │   └── export { Routes as default } from './Routes'
│   │   │
│   │   ├── 📄 Routes.tsx
│   │   │   └── export const Routes = () => (
│   │   │       ├── <RouterProvider router={createBrowserRouter([
│   │   │       │   ├── { path: '/', element: <MatchList /> },
│   │   │       │   └── { path: '/:id', element: <MatchDetail /> }
│   │   │       │   ])} />
│   │   │       └── )
│   │   │
│   │   ├── 📁 pages/
│   │   │   ├── 📄 MatchList.tsx
│   │   │   │   ├── import { useStream } from '@superbet/shared'
│   │   │   │   ├── import { matches$ } from '@superbet/shared/streams'
│   │   │   │   ├── import { VirtualTable } from '@superbet/shared/components'
│   │   │   │   └── export const MatchList: React.FC = () => {
│   │   │   │       ├── const matches = useStream(matches$, [])
│   │   │   │       ├── const [filter, setFilter] = useState<'pre' | 'live'>('pre')
│   │   │   │       ├── const filtered = useMemo(() => 
│   │   │   │       │   matches.filter(m => m.phase === filter), [matches, filter])
│   │   │   │       └── return (
│   │   │   │           ├── <div className="match-list">
│   │   │   │           │   ├── <header>
│   │   │   │           │   │   ├── <button onClick={() => setFilter('pre')}>Pre-match</button>
│   │   │   │           │   │   └── <button onClick={() => setFilter('live')}>Live</button>
│   │   │   │           │   ├── </header>
│   │   │   │           │   └── <VirtualTable data={filtered} columns={MATCH_COLUMNS} />
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 MatchDetail.tsx
│   │   │       ├── import { useParams } from 'react-router-dom'
│   │   │       ├── import { useSWR } from '@superbet/shared'
│   │   │       └── export const MatchDetail: React.FC = () => {
│   │   │           ├── const { id } = useParams()
│   │   │           ├── const match = useSWR(`/api/matches/${id}`, fetchMatch)
│   │   │           └── return (
│   │   │               ├── <div className="match-detail">
│   │   │               │   ├── <MatchTimeline match={match} />
│   │   │               │   ├── <OddsHistory match={match} />
│   │   │               │   └── <HandoverStatus match={match} />
│   │   │               └── </div>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 MatchTimeline.tsx
│   │       │   └── // SVG timeline with xG flow
│   │       └── 📄 OddsHistory.tsx
│   │           └── // Line chart with historical odds
│   │
│   └── 📁 hooks/
│       └── 📄 useMatches.ts
│           └── export const useMatches = () => useStream(matches$, [])
│
├── 📁 predictions/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 HRLRecommendations.tsx
│   │   │   │   ├── import { useSWR } from '@superbet/shared'
│   │   │   │   └── export const HRLRecommendations: React.FC = () => {
│   │   │   │       ├── const predictions = useSWR('/api/predictions', fetchPredictions)
│   │   │   │       ├── const topPicks = useMemo(() => 
│   │   │   │       │   predictions.sort((a, b) => b.vsnr - a.vsnr).slice(0, 10), [predictions])
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="HRL Top Picks">
│   │   │   │           │   └── <VirtualTable data={topPicks} columns={PREDICTION_COLUMNS} />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 CouponBuilder.tsx
│   │   │   │   └── export const CouponBuilder: React.FC = () => {
│   │   │   │       ├── const [selectedPredictions, setSelectedPredictions] = useState<string[]>([])
│   │   │   │       ├── const coupons = useSWR('/api/coupons/optimize', () => 
│   │   │   │       │   fetch('/api/coupons/optimize', {
│   │   │   │       │     method: 'POST',
│   │   │   │       │     body: JSON.stringify({ predictions: selectedPredictions })
│   │   │   │       │   }))
│   │   │   │       └── return (
│   │   │   │           ├── <div className="coupon-builder">
│   │   │   │           │   ├── <PredictionSelector onSelect={setSelectedPredictions} />
│   │   │   │           │   └── <CouponList coupons={coupons} />
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 KellyDistribution.tsx
│   │   │       └── export const KellyDistribution: React.FC = () => {
│   │   │           ├── const predictions = useSWR('/api/predictions', fetchPredictions)
│   │   │           ├── const kellyData = useMemo(() => 
│   │   │           │   predictions.map(p => ({
│   │   │           │     match_id: p.match_id,
│   │   │           │     kelly: calculateKelly(p.confidence, p.odds)
│   │   │           │   })), [predictions])
│   │   │           └── return <Card title="Kelly Pay Dağılımı"><BarChart data={kellyData} /></Card>
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 usePredictions.ts
│   │           └── export const usePredictions = () => useSWR('/api/predictions', fetchPredictions)
│   │
│   └── 📁 utils/
│       └── 📄 kelly.ts
│           └── export const calculateKelly = (prob: number, odds: number) => {
│               ├── const b = odds - 1
│               ├── const q = 1 - prob
│               └── return (b * prob - q) / b
│               }
│
├── 📁 risk/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 VaRGauge.tsx
│   │   │   │   ├── import { Gauge } from '@superbet/shared/components'
│   │   │   │   └── export const VaRGauge: React.FC = () => {
│   │   │   │       ├── const risk = useSWR('/api/risk/var', fetchVaR)
│   │   │   │       └── return <Card title="Value at Risk (95%)">
│   │   │   │           <Gauge value={risk.var} min={0} max={0.1} label="VaR" />
│   │   │   │           </Card>
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 CVaRGauge.tsx
│   │   │   │   └── // Conditional VaR gauge
│   │   │   │
│   │   │   ├── 📄 MaxDrawdown.tsx
│   │   │   │   └── // Max drawdown chart
│   │   │   │
│   │   │   └── 📄 CircuitBreakerMatrix.tsx
│   │   │       ├── import { useStream } from '@superbet/shared'
│   │   │       ├── import { circuitBreakers$ } from '@superbet/shared/streams'
│   │   │       └── export const CircuitBreakerMatrix: React.FC = () => {
│   │   │           ├── const cbs = useStream(circuitBreakers$, [])
│   │   │           └── return (
│   │   │               ├── <Card title="Circuit Breaker Matrix">
│   │   │               │   └── <table className="cb-matrix">
│   │   │               │       {cbs.map(cb => (
│   │   │               │         <tr key={cb.name}>
│   │   │               │           <td>{cb.name}</td>
│   │   │               │           <td className={`state-${cb.state.toLowerCase()}`}>{cb.state}</td>
│   │   │               │           <td>{cb.failure_count}</td>
│   │   │               │         </tr>
│   │   │               │       ))}
│   │   │               │     </table>
│   │   │               └── </Card>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useRiskMetrics.ts
│   │           └── export const useRiskMetrics = () => useSWR('/api/risk', fetchRiskMetrics)
│   │
│   └── 📁 styles/
│       └── 📄 risk.css
│           └── .cb-matrix .state-open { color: var(--accent-danger); }
│               .cb-matrix .state-closed { color: var(--accent-success); }
│
├── 📁 strategies/                 ← Analyst Dashboard
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 UCBManagerChart.tsx
│   │   │   │   ├── import { LineChart } from 'recharts'
│   │   │   │   └── export const UCBManagerChart: React.FC = () => {
│   │   │   │       ├── const strategies = useSWR('/api/strategies', fetchStrategies)
│   │   │   │       ├── const data = useMemo(() => 
│   │   │   │       │   strategies.map(s => ({
│   │   │   │       │     name: s.name,
│   │   │   │     roi: s.roi,
│   │   │   │     sharpe: s.sharpe
│   │   │   │       │   })), [strategies])
│   │   │   │       └── return <Card title="UCB Manager ROI">
│   │   │   │           <LineChart data={data} />
│   │   │   │           </Card>
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 StrategyWeightSliders.tsx
│   │   │   │   └── export const StrategyWeightSliders: React.FC = () => {
│   │   │   │       ├── const [weights, setWeights] = useState<Record<string, number>>({})
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Strateji Ağırlıkları">
│   │   │   │           │   └── <div className="weight-sliders">
│   │   │   │           │       {Object.entries(weights).map(([name, weight]) => (
│   │   │   │           │         <label key={name}>
│   │   │   │           │           {name}:
│   │   │   │           │           <input type="range" min={0} max={100} value={weight}
│   │   │   │           │             onChange={(e) => setWeights({...weights, [name]: e.target.value})} />
│   │   │   │           │         </label>
│   │   │   │           │       ))}
│   │   │   │           │     </div>
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 BCDRegimeChart.tsx
│   │   │       └── export const BCDRegimeChart: React.FC = () => {
│   │   │           ├── const regime = useSWR('/api/strategies/regime', fetchRegime)
│   │   │           └── return <Card title="BCD Rejim Algılama">
│   │   │               <LineChart data={regime.history} />
│   │   │               <p>Current p_BCD: {regime.current.toFixed(3)}</p>
│   │   │               </Card>
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useStrategies.ts
│   │           └── export const useStrategies = () => useSWR('/api/strategies', fetchStrategies)
│   │
│   └── 📁 styles/
│       └── 📄 strategies.css
│
├── 📁 backtest/                   ← Analyst (YENİ v2.1) ⭐
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 PerformanceChart.tsx
│   │   │   │   └── export const PerformanceChart: React.FC = () => {
│   │   │   │       ├── const backtest = useSWR('/api/backtest', fetchBacktest)
│   │   │   │       └── return <Card title="Tarihsel Performans">
│   │   │   │           <LineChart data={backtest.equity_curve} />
│   │   │   │           </Card>
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 StrategyComparison.tsx
│   │   │   │   └── export const StrategyComparison: React.FC = () => {
│   │   │   │       ├── const [selectedStrategies, setSelectedStrategies] = useState<string[]>([])
│   │   │   │       ├── const comparison = useSWR(
│   │   │   │       │   `/api/backtest/compare?strategies=${selectedStrategies.join(',')}`,
│   │   │   │       │   fetchComparison)
│   │   │   │       └── return <Card title="Strateji Karşılaştırma">
│   │   │   │           <table className="comparison-table">
│   │   │   │             {comparison.map(c => <tr><td>{c.name}</td><td>{c.roi}</td></tr>)}
│   │   │   │           </table>
│   │   │   │           </Card>
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 EquityCurve.tsx
│   │   │       └── // Equity curve visualization
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useBacktest.ts
│   │           └── export const useBacktest = (params) => useSWR(`/api/backtest`, () => fetchBacktest(params))
│   │
│   └── 📁 styles/
│       └── 📄 backtest.css
│
└── 📁 reports/                    ← Analyst (YENİ v2.1) ⭐
    ├── 📄 package.json
    ├── 📄 webpack.config.js
    │
    ├── 📁 src/
    │   ├── 📄 index.tsx
    │   │
    │   ├── 📁 components/
    │   │   ├── 📄 DailySummary.tsx
    │   │   │   └── export const DailySummary: React.FC = () => {
    │   │   │       ├── const summary = useSWR('/api/reports/daily', fetchDailySummary)
    │   │   │       └── return (
    │   │   │           ├── <Card title="Günlük Özet">
    │   │   │           │   ├── <p>ROI: {summary.roi}%</p>
    │   │   │           │   ├── <p>Win Rate: {summary.winRate}%</p>
    │   │   │           │   └── <p>Total Bets: {summary.totalBets}</p>
    │   │   │           └── </Card>
    │   │   │       )
    │   │   │       }
    │   │   │
    │   │   ├── 📄 WeeklyReport.tsx
    │   │   │   └── // Weekly summary with charts
    │   │   │
    │   │   └── 📄 ExportButton.tsx
    │   │       └── export const ExportButton: React.FC<{ format: 'pdf' | 'csv' }> = ({ format }) => {
    │   │           ├── const handleExport = async () => {
    │   │           │   ├── const blob = await fetch(`/api/reports/export?format=${format}`).then(r => r.blob())
    │   │           │   ├── const url = URL.createObjectURL(blob)
    │   │           │   ├── const a = document.createElement('a')
    │   │           │   ├── a.href = url
    │   │           │   ├── a.download = `report.${format}`
    │   │           │   └── a.click()
    │   │           │   }
    │   │           └── return <button onClick={handleExport}>Export {format.toUpperCase()}</button>
    │   │           }
    │   │
    │   └── 📁 utils/
    │       └── 📄 exportToPDF.ts
    │           └── export const exportToPDF = async (data) => {
    │               ├── // Use jsPDF or similar library
    │               └── return blob
    │               }
    │
    └── 📁 styles/
        └── 📄 reports.css
```

---

## 📊 BÖLÜM B İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Modüller** | 7 (signal-center, matches, predictions, risk, strategies, backtest, reports) |
| **Toplam Dosya** | 45+ |
| **React Bileşenleri** | 30+ |
| **Hooks** | 7 |
| **Rol Bazlı** | Operator: 4 modül, Analyst: 3 modül |

---

## 🔗 DEVAMI

**Sonraki Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_C.md`  
**İçerik:** DevOps Modülleri (Metrics, **Logs ⭐**, Traces, Alerts, Health)

---

**Versiyon:** SUPERBET GENESIS v3.1 Frontend Tree - PART 6B  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md
