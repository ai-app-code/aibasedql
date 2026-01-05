# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 6A
## Frontend - Shell + Shared Infrastructure (4+ Seviye Derinlik)

**Tarih:** 05.01.2026 | **Referans:** FRONTEND_ARCHITECTURE_v2.1.md

---

```
superbet-genesis/ (devam)
│
└── 📁 frontend/
    │
    ├── 📁 shell/
    │   ├── 📄 package.json
    │   │   ├── name: "@superbet/shell"
    │   │   ├── dependencies:
    │   │   │   ├── react: ^18.2.0
    │   │   │   ├── react-dom: ^18.2.0
    │   │   │   ├── react-router-dom: ^6.20.0
    │   │   │   ├── rxjs: ^7.8.0
    │   │   │   └── @module-federation/webpack: ^2.0.0
    │   │   └── scripts:
    │   │       ├── dev: webpack serve --mode development
    │   │       └── build: webpack --mode production
    │   │
    │   ├── 📄 webpack.config.js
    │   │   ├── const { ModuleFederationPlugin } = require('webpack').container
    │   │   ├── module.exports:
    │   │   │   ├── entry: './src/index.tsx'
    │   │   │   ├── output:
    │   │   │   │   ├── path: dist/
    │   │   │   │   └── publicPath: 'auto'
    │   │   │   └── plugins:
    │   │   │       └── ModuleFederationPlugin:
    │   │   │           ├── name: 'shell'
    │   │   │           ├── remotes:
    │   │   │           │   ├── signalCenter: 'signalCenter@https://cdn.superbet/signal-center/remoteEntry.js'
    │   │   │           │   ├── matches: 'matches@https://cdn.superbet/matches/remoteEntry.js'
    │   │   │           │   ├── risk: 'risk@https://cdn.superbet/risk/remoteEntry.js'
    │   │   │           │   ├── strategies: 'strategies@https://cdn.superbet/strategies/remoteEntry.js'
    │   │   │           │   ├── logs: 'logs@https://cdn.superbet/logs/remoteEntry.js'
    │   │   │           │   ├── terminal: 'terminal@https://cdn.superbet/terminal/remoteEntry.js'
    │   │   │           │   ├── mlops: 'mlops@https://cdn.superbet/mlops/remoteEntry.js'
    │   │   │           │   └── admin: 'admin@https://cdn.superbet/admin/remoteEntry.js'
    │   │   │           └── shared:
    │   │   │               ├── react: { singleton: true, requiredVersion: '^18.2.0' }
    │   │   │               ├── react-dom: { singleton: true, requiredVersion: '^18.2.0' }
    │   │   │               ├── rxjs: { singleton: true, requiredVersion: '^7.8.0' }
    │   │   │               ├── react-window: { singleton: true }
    │   │   │               └── xterm: { singleton: true }
    │   │
    │   ├── 📁 src/
    │   │   ├── 📄 index.tsx
    │   │   │   ├── import ReactDOM from 'react-dom/client'
    │   │   │   ├── import { App } from './App'
    │   │   │   ├── const root = ReactDOM.createRoot(document.getElementById('root'))
    │   │   │   └── root.render(<App />)
    │   │   │
    │   │   ├── 📄 App.tsx
    │   │   │   ├── import { BrowserRouter } from 'react-router-dom'
    │   │   │   ├── import { RoleProvider, AuthProvider, ThemeProvider } from './providers'
    │   │   │   ├── import { ShellLayout } from './components/ShellLayout'
    │   │   │   └── export const App = () => (
    │   │   │       ├── <AuthProvider>
    │   │   │       │   └── <RoleProvider>
    │   │   │       │       └── <ThemeProvider>
    │   │   │       │           └── <BrowserRouter>
    │   │   │       │               └── <ShellLayout />
    │   │   │       │           └── </BrowserRouter>
    │   │   │       │       └── </ThemeProvider>
    │   │   │       │   └── </RoleProvider>
    │   │   │       └── </AuthProvider>
    │   │   │   )
    │   │   │
    │   │   ├── 📁 components/
    │   │   │   ├── 📄 ShellLayout.tsx
    │   │   │   │   ├── import { Header, Sidebar, Footer } from '.'
    │   │   │   │   ├── import { Routes, Route } from 'react-router-dom'
    │   │   │   │   ├── import { lazy } from 'react'
    │   │   │   │   ├── const SignalCenter = lazy(() => import('signalCenter/RemoteEntry'))
    │   │   │   │   ├── const Matches = lazy(() => import('matches/RemoteEntry'))
    │   │   │   │   ├── const Logs = lazy(() => import('logs/RemoteEntry'))
    │   │   │   │   ├── const Terminal = lazy(() => import('terminal/RemoteEntry'))
    │   │   │   │   └── return (
    │   │   │   │       ├── <div className="shell-layout">
    │   │   │   │       │   ├── <Header />
    │   │   │   │       │   ├── <div className="main-container">
    │   │   │   │       │   │   ├── <Sidebar />
    │   │   │   │       │   │   └── <main className="content-area">
    │   │   │   │       │   │       └── <Suspense fallback={<Skeleton />}>
    │   │   │   │       │   │           └── <Routes>
    │   │   │   │       │   │               ├── <Route path="/operator/*" element={<SignalCenter />} />
    │   │   │   │       │   │               ├── <Route path="/devops/logs" element={<Logs />} />
    │   │   │   │       │   │               ├── <Route path="/infra/terminal" element={<Terminal />} />
    │   │   │   │       │   │               └── {/* ... */}
    │   │   │   │       │   │           └── </Routes>
    │   │   │   │       │   │       └── </Suspense>
    │   │   │   │       │   │   └── </main>
    │   │   │   │       │   ├── </div>
    │   │   │   │       │   └── <Footer />
    │   │   │   │       └── </div>
    │   │   │   │   )
    │   │   │   │
    │   │   │   ├── 📄 Header.tsx
    │   │   │   │   ├── export const Header: React.FC = () => {
    │   │   │   │   │   ├── const { currentRole, setRole } = useRole()
    │   │   │   │   │   ├── const wsStatus = useStream(wsStatus$)
    │   │   │   │   │   ├── const kafkaLag = useStream(metrics$.pipe(map(m => m.kafkaLag)))
    │   │   │   │   │   └── return (
    │   │   │   │   │       ├── <header className="shell-header">
    │   │   │   │   │       │   ├── <div className="logo">
    │   │   │   │   │       │   │   └── SUPERBET GENESIS v3.1
    │   │   │   │   │       │   │   </div>
    │   │   │   │   │       │   ├── <RoleTabs>
    │   │   │   │   │       │   │   ├── ['Operator', 'Analyst', 'DevOps', 'Infra', 'MLOps', 'Admin'].map(role =>
    │   │   │   │   │       │   │   │   ├── <Tab 
    │   │   │   │   │       │   │   │   │   key={role} 
    │   │   │   │   │       │   │   │   │   active={currentRole === role}
    │   │   │   │   │       │   │   │   │   onClick={() => setRole(role)}
    │   │   │   │   │       │   │   │   │   >
    │   │   │   │   │       │   │   │   │   {role}
    │   │   │   │   │       │   │   │   └── </Tab>
    │   │   │   │   │       │   │   └── )
    │   │   │   │   │       │   ├── </RoleTabs>
    │   │   │   │   │       │   ├── <SystemStatus>
    │   │   │   │   │       │   │   ├── WebSocket: {wsStatus.connected ? '●' : '○'}
    │   │   │   │   │       │   │   ├── Kafka Lag: {kafkaLag}ms
    │   │   │   │   │       │   │   └── p99: {wsStatus.p99}ms
    │   │   │   │   │       │   ├── </SystemStatus>
    │   │   │   │   │       │   ├── <GlobalSearch />
    │   │   │   │   │       │   ├── <Alerts />
    │   │   │   │   │       │   └── <UserMenu />
    │   │   │   │   │       └── </header>
    │   │   │   │   │   )
    │   │   │   │   │   }
    │   │   │   │   │
    │   │   │   │   └── HEADER_HEIGHT = 60 // px
    │   │   │   │
    │   │   │   ├── 📄 Sidebar.tsx
    │   │   │   │   ├── export const Sidebar: React.FC = () => {
    │   │   │   │   │   ├── const { currentRole } = useRole()
    │   │   │   │   │   ├── const [collapsed, setCollapsed] = useState(false)
    │   │   │   │   │   ├── const menuItems = useMemo(() => {
    │   │   │   │   │   │   └── switch (currentRole) {
    │   │   │   │   │   │       ├── case 'Operator': return OPERATOR_MENU
    │   │   │   │   │   │       ├── case 'DevOps': return DEVOPS_MENU
    │   │   │   │   │   │       ├── case 'Infra': return INFRA_MENU
    │   │   │   │   │   │       └── // ...
    │   │   │   │   │   │   }
    │   │   │   │   │   │   }, [currentRole])
    │   │   │   │   │   └── return (
    │   │   │   │   │       ├── <aside className={`sidebar ${collapsed ? 'collapsed' : ''}`}>
    │   │   │   │   │       │   ├── <nav>
    │   │   │   │   │       │   │   └── {menuItems.map(item => <MenuItem key={item.path} {...item} />)}
    │   │   │   │   │       │   └── </nav>
    │   │   │   │   │       │   └── <CollapseToggle onClick={() => setCollapsed(!collapsed)} />
    │   │   │   │   │       └── </aside>
    │   │   │   │   │   )
    │   │   │   │   │   }
    │   │   │   │   │
    │   │   │   │   ├── SIDEBAR_WIDTH = 280 // px (expanded)
    │   │   │   │   └── SIDEBAR_WIDTH_COLLAPSED = 64 // px
    │   │   │   │
    │   │   │   └── 📄 Footer.tsx
    │   │   │       ├── export const Footer: React.FC = () => {
    │   │   │       │   ├── const buildInfo = useBuildInfo()
    │   │   │       │   └── return (
    │   │   │       │       ├── <footer className="shell-footer">
    │   │   │       │       │   ├── Build: {buildInfo.commit.slice(0, 7)}
    │   │   │       │       │   ├── Model: {buildInfo.modelVersion}
    │   │   │       │       │   ├── Canary: {buildInfo.canaryPercentage}%
    │   │   │       │       │   └── Env: {buildInfo.environment}
    │   │   │       │       └── </footer>
    │   │   │       │   )
    │   │   │       │   }
    │   │   │       └── FOOTER_HEIGHT = 40 // px (non-sticky)
    │   │   │
    │   │   ├── 📁 providers/
    │   │   │   ├── 📄 AuthProvider.tsx
    │   │   │   │   └── export const AuthProvider: React.FC<PropsWithChildren> = ({ children }) => {
    │   │   │   │       ├── const [user, setUser] = useState<User | null>(null)
    │   │   │   │       ├── useEffect(() => {
    │   │   │   │       │   └── // Fetch user from JWT/mTLS
    │   │   │   │       │   }, [])
    │   │   │   │       └── return <AuthContext.Provider value={{ user }}>{children}</AuthContext.Provider>
    │   │   │   │       }
    │   │   │   │
    │   │   │   ├── 📄 RoleProvider.tsx
    │   │   │   │   └── export const RoleProvider: React.FC<PropsWithChildren> = ({ children }) => {
    │   │   │   │       ├── const [currentRole, setCurrentRole] = useState<Role>('Operator')
    │   │   │   │       ├── useEffect(() => {
    │   │   │   │       │   ├── const saved = localStorage.getItem('superbet:currentRole')
    │   │   │   │       │   └── if (saved) setCurrentRole(saved as Role)
    │   │   │   │       │   }, [])
    │   │   │   │       ├── const setRole = useCallback((role: Role) => {
    │   │   │   │       │   ├── setCurrentRole(role)
    │   │   │   │       │   └── localStorage.setItem('superbet:currentRole', role)
    │   │   │   │       │   }, [])
    │   │   │   │       └── return <RoleContext.Provider value={{ currentRole, setRole }}>{children}</RoleContext.Provider>
    │   │   │   │       }
    │   │   │   │
    │   │   │   └── 📄 ThemeProvider.tsx
    │   │   │       └── export const ThemeProvider: React.FC<PropsWithChildren> = ({ children }) => {
    │   │   │           ├── const [theme, setTheme] = useState<'dark' | 'light'>('dark')
    │   │   │           └── // Apply CSS variables based on theme
    │   │   │           }
    │   │   │
    │   │   └── 📁 hooks/
    │   │       ├── 📄 useRole.ts
    │   │       │   └── export const useRole = () => useContext(RoleContext)
    │   │       ├── 📄 useAuth.ts
    │   │       │   └── export const useAuth = () => useContext(AuthContext)
    │   │       └── 📄 useBuildInfo.ts
    │   │           └── export const useBuildInfo = () => useSWR('/api/build-info', fetcher)
    │   │
    │   └── 📁 public/
    │       ├── 📄 index.html
    │       └── 📁 assets/
    │           └── logo.svg
    │
    ├── 📁 shared/
    │   ├── 📄 package.json
    │   │   ├── name: "@superbet/shared"
    │   │   └── dependencies:
    │   │       ├── react: ^18.2.0
    │   │       ├── rxjs: ^7.8.0
    │   │       ├── react-window: ^3.0.0
    │   │       ├── deck.gl: ^8.9.0
    │   │       └── protobufjs: ^7.2.0
    │   │
    │   ├── 📁 components/
    │   │   ├── 📄 Card.tsx
    │   │   │   └── export const Card: React.FC<CardProps> = ({ children, title, ...props }) => (
    │   │   │       ├── <div className="card">
    │   │   │       │   ├── {title && <h3 className="card-title">{title}</h3>}
    │   │   │       │   └── <div className="card-content">{children}</div>
    │   │   │       └── </div>
    │   │   │       )
    │   │   │
    │   │   ├── 📄 Gauge.tsx
    │   │   │   └── export const Gauge: React.FC<GaugeProps> = ({ value, min, max, label }) => {
    │   │   │       ├── const percentage = ((value - min) / (max - min)) * 100
    │   │   │       └── return (
    │   │   │           ├── <svg viewBox="0 0 100 100" className="gauge">
    │   │   │           │   ├── <circle cx="50" cy="50" r="40" className="gauge-bg" />
    │   │   │           │   ├── <circle cx="50" cy="50" r="40" className="gauge-fill"
    │   │   │           │   │   strokeDasharray={`${percentage * 2.51} 251`}
    │   │   │           │   │   />
    │   │   │           │   └── <text x="50" y="55" textAnchor="middle">{value}</text>
    │   │   │           └── </svg>
    │   │   │       )
    │   │   │       }
    │   │   │
    │   │   ├── 📄 Donut.tsx
    │   │   │   └── export const Donut: React.FC<DonutProps> = ({ data }) => {
    │   │   │       ├── const total = data.reduce((sum, d) => sum + d.value, 0)
    │   │   │       └── // SVG donut chart with segments
    │   │   │       }
    │   │   │
    │   │   ├── 📄 Sparkline.tsx
    │   │   │   └── export const Sparkline: React.FC<SparklineProps> = ({ data }) => {
    │   │   │       ├── const path = useMemo(() => {
    │   │   │       │   └── // Generate SVG path from data points
    │   │   │       │   }, [data])
    │   │   │       └── return <svg><path d={path} /></svg>
    │   │   │       }
    │   │   │
    │   │   ├── 📄 VirtualTable.tsx
    │   │   │   ├── import { FixedSizeList as List } from 'react-window'
    │   │   │   ├── import AutoSizer from 'react-virtualized-auto-sizer'
    │   │   │   └── export const VirtualTable: React.FC<VirtualTableProps> = ({ data, columns }) => {
    │   │   │       ├── const Row = ({ index, style }) => {
    │   │   │       │   └── <div style={style} className="table-row">
    │   │   │       │       └── {columns.map(col => <Cell key={col.key} value={data[index][col.key]} />)}
    │   │   │       │       </div>
    │   │   │       │   }
    │   │   │       └── return (
    │   │   │           ├── <AutoSizer>
    │   │   │           │   └── {({ height, width }) => (
    │   │   │           │       ├── <List
    │   │   │           │       │   height={height}
    │   │   │           │       │   width={width}
    │   │   │           │       │   itemCount={data.length}
    │   │   │           │       │   itemSize={48}
    │   │   │           │       │   overscanCount={5}
    │   │   │           │       │   >
    │   │   │           │       │   {Row}
    │   │   │           │       └── </List>
    │   │   │           │   )}
    │   │   │           └── </AutoSizer>
    │   │   │       )
    │   │   │       }
    │   │   │
    │   │   ├── 📄 Heatmap.tsx
    │   │   │   ├── import { DeckGL } from '@deck.gl/react'
    │   │   │   ├── import { HeatmapLayer } from '@deck.gl/aggregation-layers'
    │   │   │   └── export const Heatmap: React.FC<HeatmapProps> = ({ data }) => {
    │   │   │       ├── const layer = new HeatmapLayer({
    │   │   │       │   ├── id: 'heatmap',
    │   │   │       │   ├── data,
    │   │   │       │   ├── getPosition: d => [d.x, d.y],
    │   │   │       │   └── getWeight: d => d.value
    │   │   │       │   })
    │   │   │       └── return <DeckGL layers={[layer]} viewState={viewState} />
    │   │   │       }
    │   │   │
    │   │   └── 📄 AlertBanner.tsx
    │   │       └── export const AlertBanner: React.FC<AlertBannerProps> = ({ type, message }) => (
    │   │           ├── <div className={`alert alert-${type}`}>
    │   │           │   └── {message}
    │   │           └── </div>
    │   │           )
    │   │
    │   ├── 📁 hooks/
    │   │   ├── 📄 useStream.ts
    │   │   │   ├── import { useSyncExternalStore, useCallback } from 'react'
    │   │   │   ├── import { Observable } from 'rxjs'
    │   │   │   └── export function useStream<T>(stream$: Observable<T>, initialValue: T): T {
    │   │   │       ├── const subscribe = useCallback((callback: () => void) => {
    │   │   │       │   ├── const sub = stream$.subscribe(callback)
    │   │   │       │   └── return () => sub.unsubscribe()
    │   │   │       │   }, [stream$])
    │   │   │       ├── const getSnapshot = useCallback(() => {
    │   │   │       │   └── return streamStore.get(stream$) ?? initialValue
    │   │   │       │   }, [stream$, initialValue])
    │   │   │       └── return useSyncExternalStore(subscribe, getSnapshot)
    │   │   │       }
    │   │   │
    │   │   ├── 📄 useSWR.ts
    │   │   │   ├── import { BehaviorSubject, merge, of, from } from 'rxjs'
    │   │   │   ├── import { tap } from 'rxjs/operators'
    │   │   │   ├── import { openDB } from 'idb'
    │   │   │   └── export function useSWR<T>(key: string, fetcher: () => Promise<T>) {
    │   │   │       ├── const cache$ = useMemo(() => new BehaviorSubject<T | null>(null), [key])
    │   │   │       ├── useEffect(() => {
    │   │   │       │   ├── // Try IndexedDB first
    │   │   │       │   ├── const loadCached = async () => {
    │   │   │       │   │   ├── const db = await openDB('superbet-cache', 1)
    │   │   │       │   │   └── const cached = await db.get('cache', key)
    │   │   │       │   │   if (cached) cache$.next(cached)
    │   │   │       │   │   }
    │   │   │       │   ├── loadCached()
    │   │   │       │   └── // Fetch fresh data
    │   │   │       │       fetcher().then(data => {
    │   │   │       │           ├── cache$.next(data)
    │   │   │       │           └── // Save to IndexedDB
    │   │   │       │           })
    │   │   │       │   }, [key])
    │   │   │       └── return useStream(cache$, null)
    │   │   │       }
    │   │   │
    │   │   └── 📄 usePermission.ts
    │   │       ├── import { useAuth } from '@superbet/shell'
    │   │       ├── import { PERMISSION_MATRIX } from '../config/rbac'
    │   │       └── export const usePermission = (resource: string, action: 'read' | 'write' | 'execute') => {
    │   │           ├── const { user } = useAuth()
    │   │           ├── const role = user?.role ?? 'operator'
    │   │           └── return PERMISSION_MATRIX[role]?.[resource]?.[action] ?? false
    │   │           }
    │   │
    │   ├── 📁 streams/
    │   │   ├── 📄 index.ts
    │   │   │   ├── import { Subject } from 'rxjs'
    │   │   │   ├── import { map, shareReplay } from 'rxjs/operators'
    │   │   │   ├── import { decode } from 'protobufjs'
    │   │   │   ├── import { TwinDelta } from '../proto/superbet'
    │   │   │   ├── // BroadcastChannel bridge from SharedWorker
    │   │   │   ├── const channel = new BroadcastChannel('superbet-realtime')
    │   │   │   ├── const rawBinary$ = new Subject<ArrayBuffer>()
    │   │   │   ├── channel.onmessage = (e) => rawBinary$.next(e.data)
    │   │   │   ├── // Decode Protobuf
    │   │   │   ├── export const decoded$ = rawBinary$.pipe(
    │   │   │   │   ├── map(buffer => decode(TwinDelta, new Uint8Array(buffer))),
    │   │   │   │   └── shareReplay({ bufferSize: 50, windowTime: 60000 })
    │   │   │   │   )
    │   │   │   └── export * from './alarms'
    │   │   │       export * from './matches'
    │   │   │       export * from './broadcast'
    │   │   │
    │   │   ├── 📄 alarms.ts
    │   │   │   ├── import { decoded$ } from './index'
    │   │   │   ├── import { filter, bufferTime, map } from 'rxjs/operators'
    │   │   │   └── export const alarms$ = decoded$.pipe(
    │   │   │       ├── filter(msg => msg.type === 'alarm'),
    │   │   │       ├── bufferTime(250),
    │   │   │       └── map(batch => batch.slice(-100)) // Keep last 100
    │   │   │       )
    │   │   │
    │   │   ├── 📄 matches.ts
    │   │   │   ├── import { decoded$ } from './index'
    │   │   │   ├── import { filter, map, throttleTime } from 'rxjs/operators'
    │   │   │   └── export const matches$ = decoded$.pipe(
    │   │   │       ├── filter(msg => msg.type === 'match'),
    │   │   │       ├── map(groupByMatchId),
    │   │   │       └── throttleTime(100)
    │   │   │       )
    │   │   │
    │   │   ├── 📄 broadcast.ts
    │   │   │   ├── import { decoded$ } from './index'
    │   │   │   ├── import { filter, map } from 'rxjs/operators'
    │   │   │   └── export const broadcast$ = decoded$.pipe(
    │   │   │       ├── filter(msg => msg.type === 'broadcast'),
    │   │   │       └── map(event => ({
    │   │   │           ├── ...event,
    │   │   │           ├── isFiltered: event.metrics.confidence > 0.65 &&
    │   │   │           │              event.metrics.vsnr > 2.2 &&
    │   │   │           │              event.metrics.cas > 1.0,
    │   │   │           └── isLocked: event.metrics.uncertainty > 0.4
    │   │   │           }))
    │   │   │       )
    │   │   │
    │   │   ├── 📄 agents.ts
    │   │   │   └── export const agents$ = decoded$.pipe(filter(msg => msg.type === 'agent_status'))
    │   │   │
    │   │   └── 📄 circuitBreakers.ts
    │   │       └── export const circuitBreakers$ = decoded$.pipe(filter(msg => msg.type === 'cb_status'))
    │   │
    │   ├── 📁 utils/
    │   │   ├── 📄 protobuf.ts
    │   │   │   ├── import { load } from 'protobufjs'
    │   │   │   ├── let root: protobuf.Root
    │   │   │   ├── export async function initProto() {
    │   │   │   │   └── root = await load('/proto/superbet.proto')
    │   │   │   │   }
    │   │   │   └── export const decode = (MessageType, buffer) => root.lookupType(MessageType).decode(buffer)
    │   │   │
    │   │   └── 📄 formatting.ts
    │   │       ├── export const formatCurrency = (value: number) => `€${value.toFixed(2)}`
    │   │       ├── export const formatOdds = (odds: number) => odds.toFixed(2)
    │   │       └── export const formatTimestamp = (ts: number) => new Date(ts).toISOString()
    │   │
    │   └── 📁 config/
    │       └── 📄 rbac.ts
    │           └── export const PERMISSION_MATRIX = {
    │               ├── operator: {
    │               │   ├── signals: { read: true, write: false, execute: false },
    │               │   ├── matches: { read: true, write: false, execute: false },
    │               │   ├── predictions: { read: true, write: false, execute: false },
    │               │   └── risk: { read: true, write: false, execute: false }
    │               │   },
    │               ├── devops: {
    │               │   ├── logs: { read: true, write: false, execute: false },
    │               │   ├── metrics: { read: true, write: false, execute: false },
    │               │   └── alerts: { read: true, write: true, execute: false }
    │               │   },
    │               ├── infra: {
    │               │   ├── k8s: { read: true, write: true, execute: true },
    │               │   ├── terminal: { read: true, write: true, execute: true },
    │               │   └── config: { read: true, write: true, execute: false }
    │               │   },
    │               └── admin: {
    │                   └── '*': { read: true, write: true, execute: true }
    │                   }
    │               }
    │
    └── 📁 worker/
        ├── 📄 package.json
        │   ├── name: "@superbet/worker"
        │   └── dependencies: {}
        │
        ├── 📄 shared.ts
        │   ├── const sockets = new Map<string, WebSocket>()
        │   ├── const channel = new BroadcastChannel('superbet-realtime')
        │   ├── self.onconnect = (e: MessageEvent) => {
        │   │   ├── const port = e.ports[0]
        │   │   ├── // Tek WS bağlantısı - tüm sekmeler paylaşır
        │   │   ├── if (!sockets.has('main')) {
        │   │   │   ├── const ws = new WebSocket('wss://bff.superbet:8080/ws/alarms')
        │   │   │   ├── ws.binaryType = 'arraybuffer'
        │   │   │   ├── ws.onmessage = (event) => {
        │   │   │   │   └── // BroadcastChannel ile tüm sekmelere yayınla
        │   │   │   │       channel.postMessage(event.data)
        │   │   │   │   }
        │   │   │   ├── ws.onclose = () => {
        │   │   │   │   ├── sockets.delete('main')
        │   │   │   │   └── setTimeout(() => reconnect(), 1000)
        │   │   │   │   }
        │   │   │   └── sockets.set('main', ws)
        │   │   │   }
        │   │   └── port.start()
        │   │   }
        │   │
        │   └── function reconnect(attempt = 0) {
        │       ├── const delay = Math.min(1000 * Math.pow(2, attempt), 30000)
        │       └── setTimeout(() => {
        │           ├── try {
        │           │   ├── const ws = new WebSocket('wss://bff.superbet:8080/ws/alarms')
        │           │   └── sockets.set('main', ws)
        │           │   } catch {
        │           │       └── reconnect(attempt + 1)
        │           │   }
        │           }, delay)
        │       }
        │
        └── 📄 types.ts
            └── export interface WorkerMessage {
                ├── type: 'subscribe' | 'unsubscribe'
                └── channel: string
                }
```

---

## 📊 BÖLÜM A İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Ana Klasörler** | 3 (shell, shared, worker) |
| **Toplam Dosya** | 40+ |
| **React Bileşenleri** | 15+ |
| **Hooks** | 7 |
| **RxJS Streams** | 6 |
| **Utilities** | 3 |

---

## 🔗 DEVAMI

**Sonraki Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_B.md`  
**İçerik:** Operator + Analyst Modülleri (Signal Center, Matches, Predictions, Risk, Strategies, Backtest, Reports)

---

**Versiyon:** SUPERBET GENESIS v3.1 Frontend Tree - PART 6A  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md
