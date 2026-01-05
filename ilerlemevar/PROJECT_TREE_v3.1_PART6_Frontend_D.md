# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 6D
## Frontend - Infrastructure Modülleri (Terminal ⭐⭐⭐) (4+ Seviye Derinlik)

**Tarih:** 05.01.2026 | **Referans:** FRONTEND_ARCHITECTURE_v2.1.md

---

```
superbet-genesis/frontend/modules/ (devam)
│
├── 📁 k8s/                        ← Infrastructure Dashboard
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       ├── @superbet/shared: workspace:*
│   │       └── react-table: ^8.10.0
│   │
│   ├── 📄 webpack.config.js
│   │   └── exposes: { './RemoteEntry': './src/index.tsx' }
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 ClusterOverview.tsx
│   │   │   │   └── export const ClusterOverview: React.FC = () => {
│   │   │   │       ├── const cluster = useSWR('/api/k8s/cluster', fetchClusterInfo)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Cluster Overview">
│   │   │   │           │   ├── <p>Name: {cluster.name}</p>
│   │   │   │           │   ├── <p>Version: {cluster.version}</p>
│   │   │   │           │   ├── <p>Nodes: {cluster.nodeCount}</p>
│   │   │   │           │   └── <Gauge value={cluster.cpuUsage} label="CPU Usage" />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 PodsTable.tsx
│   │   │   │   └── export const PodsTable: React.FC = () => {
│   │   │   │       ├── const pods = useSWR('/api/k8s/pods', fetchPods)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Pods">
│   │   │   │           │   └── <VirtualTable
│   │   │   │           │       data={pods}
│   │   │   │           │       columns={[
│   │   │   │           │         { key: 'name', label: 'Name' },
│   │   │   │           │         { key: 'namespace', label: 'Namespace' },
│   ││   │   │           │         { key: 'status', label: 'Status' },
│   │   │   │           │         { key: 'cpu', label: 'CPU' },
│   │   │   │           │         { key: 'memory', label: 'Memory' }
│   │   │   │           │       ]}
│   │   │   │           │     />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 DeploymentsTable.tsx
│   │   │   │   └── // Deployments table
│   │   │   │
│   │   │   └── 📄 HPAStatus.tsx
│   │   │       └── export const HPAStatus: React.FC = () => {
│   │   │           ├── const hpas = useSWR('/api/k8s/hpa', fetchHPAs)
│   │   │           └── return (
│   │   │               ├── <Card title="Horizontal Pod Autoscalers">
│   │   │               │   └── {hpas.map(hpa => (
│   │   │               │       <div key={hpa.name} className="hpa-row">
│   │   │               │         <span>{hpa.name}</span>
│   │   │               │         <Sparkline data={hpa.replicaHistory} />
│   │   │               │         <span>{hpa.currentReplicas}/{hpa.desiredReplicas}</span>
│   │   │               │       </div>
│   │   │               │     ))}
│   │   │               └── </Card>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useK8s.ts
│   │           └── export const useK8s = (resource: string) => useSWR(`/api/k8s/${resource}`, fetchK8s)
│   │
│   └── 📁 styles/
│       └── 📄 k8s.css
│
├── 📁 argocd/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 ApplicationsList.tsx
│   │       │   └── export const ApplicationsList: React.FC = () => {
│   │       │       ├── const apps = useSWR('/api/argocd/applications', fetchArgoApps)
│   │       │       └── return (
│   │       │           ├── <Card title="ArgoCD Applications">
│   │       │           │   └── <VirtualTable data={apps} columns={APP_COLUMNS} />
│   │       │           └── </Card>
│   │       │       )
│   │       │       }
│   │       │
│   │       ├── 📄 SyncStatus.tsx
│   │       │   └── // Sync status indicators
│   │       │
│   │       ├── 📄 CanaryProgress.tsx
│   │       │   └── export const CanaryProgress: React.FC<{ app: string }> = ({ app }) => {
│   │       │       ├── const canary = useSWR(`/api/argocd/canary/${app}`, fetchCanaryStatus)
│   │       │       └── return (
│   │       │           ├── <Card title="Canary Deployment">
│   │       │           │   ├── <div className="canary-bar">
│   │       │           │   │   <div 
│   │       │           │   │     className="canary-fill" 
│   │       │           │   │     style={{ width: `${canary.percentage}%` }}
│   │       │           │   │   />
│   │       │           │   └── </div>
│   │       │           │   └── <p>{canary.percentage}% traffic to new version</p>
│   │       │           └── </Card>
│   │       │       )
│   │       │       }
│   │       │
│   │       └── 📄 RollbackButton.tsx
│   │           └── export const RollbackButton: React.FC<{ app: string }> = ({ app }) => {
│   │               ├── const rollback = async () => {
│   │               │   └── await fetch(`/api/argocd/rollback/${app}`, { method: 'POST' })
│   │               │   }
│   │               └── return <button onClick={rollback}>Rollback</button>
│   │               }
│   │
│   └── 📁 styles/
│       └── 📄 argocd.css
│
├── 📁 terminal/                   ← Infrastructure (KRİTİK YENİ v2.1) ⭐⭐⭐
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       ├── @superbet/shared: workspace:*
│   │       ├── xterm: ^5.3.0
│   │       ├── xterm-addon-fit: ^0.8.0
│   │       └── xterm-addon-web-links: ^0.9.0
│   │
│   ├── 📄 webpack.config.js
│   │   └── ModuleFederationPlugin:
│   │       ├── name: 'terminal'
│   │       ├── exposes: { './RemoteEntry': './src/index.tsx' }
│   │       └── shared: ['react', 'xterm']
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │   └── export { TerminalModule as default } from './TerminalModule'
│   │   │
│   │   ├── 📄 TerminalModule.tsx
│   │   │   └── export const TerminalModule: React.FC = () => {
│   │   │       ├── return (
│   │   │       │   ├── <div className="terminal-module">
│   │   │       │   │   ├── <TerminalHeader>
│   │   │       │   │   │   ├── <PodSelector />
│   │   │       │   │   │   ├── <TerminalTabs />
│   │   │       │   │   │   ├── <ReadOnlyToggle />
│   │   │       │   │   │   └── <SessionInfo />
│   │   │       │   │   ├── </TerminalHeader>
│   │   │       │   │   └── <Terminal />
│   │   │       │   └── </div>
│   │   │       └── )
│   │   │       }
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 Terminal.tsx
│   │   │   │   ├── import { Terminal as XTerm } from 'xterm'
│   │   │   │   ├── import { FitAddon } from 'xterm-addon-fit'
│   │   │   │   ├── import { WebLinksAddon } from 'xterm-addon-web-links'
│   │   │   │   ├── import { useTerminal } from '../hooks/useTerminal'
│   │   │   │   └── export const Terminal: React.FC = () => {
│   │   │   │       ├── const terminalRef = useRef<HTMLDivElement>(null)
│   │   │   │       ├── const xtermRef = useRef<XTerm | null>(null)
│   │   │   │       ├── const { send, receive, connect, disconnect, isReadOnly } = useTerminal()
│   │   │   │       ├── useEffect(() => {
│   │   │   │       │   ├── if (!terminalRef.current) return
│   │   │   │       │   ├── // Initialize xterm.js
│   │   │   │       │   ├── const xterm = new XTerm({
│   │   │   │       │   │   ├── cursorBlink: true,
│   │   │   │       │   │   ├── fontSize: 14,
│   │   │   │       │   │   ├── fontFamily: 'JetBrains Mono, monospace',
│   │   │   │       │   │   ├── theme: {
│   │   │   │       │   │   │   ├── background: '#0A0A0F',
│   │   │   │       │   │   │   ├── foreground: '#FFFFFF',
│   │   │   │       │   │   │   ├── cursor: '#00D4FF',
│   │   │   │       │   │   │   └── selection: 'rgba(79, 70, 229, 0.3)'
│   │   │   │       │   │   │   },
│   │   │   │       │   │   └── disableStdin: false
│   │   │   │       │   │   })
│   │   │   │       │   ├── // Addons
│   │   │   │       │   ├── const fitAddon = new FitAddon()
│   │   │   │       │   ├── const webLinksAddon = new WebLinksAddon()
│   │   │   │       │   ├── xterm.loadAddon(fitAddon)
│   │   │   │       │   ├── xterm.loadAddon(webLinksAddon)
│   │   │   │       │   ├── // Mount
│   │   │   │       │   ├── xterm.open(terminalRef.current)
│   │   │   │       │   ├── fitAddon.fit()
│   │   │   │       │   ├── // Handle input (send to WebSocket)
│   │   │   │       │   ├── xterm.onData(data => {
│   │   │   │       │   │   ├── if (!isReadOnly) {
│   │   │   │       │   │   │   └── send(data)
│   │   │   │       │   │   │   }
│   │   │   │       │   │   })
│   │   │   │       │   ├── // Handle output (from WebSocket)
│   │   │   │       │   ├── receive((data: ArrayBuffer) => {
│   │   │   │       │   │   ├── const decoder = new TextDecoder()
│   │   │   │       │   │   └── xterm.write(decoder.decode(data))
│   │   │   │       │   │   })
│   │   │   │       │   ├── // Resize handling
│   │   │   │       │   ├── const handleResize = () => {
│   │   │   │       │   │   ├── fitAddon.fit()
│   │   │   │       │   │   └── send(JSON.stringify({ type: 'resize', cols: xterm.cols, rows: xterm.rows }))
│   │   │   │       │   │   }
│   │   │   │       │   ├── window.addEventListener('resize', handleResize)
│   │   │   │       │   ├── xtermRef.current = xterm
│   │   │   │       │   └── return () => {
│   │   │   │       │       ├── xterm.dispose()
│   │   │   │       │       ├── window.removeEventListener('resize', handleResize)
│   │   │   │       │       └── disconnect()
│   │   │   │       │       }
│   │   │   │       │   }, [])
│   │   │   │       ├── // Update read-only state
│   │   │   │       ├── useEffect(() => {
│   │   │   │       │   ├── if (xtermRef.current) {
│   │   │   │       │   │   └── xtermRef.current.options.disableStdin = isReadOnly
│   │   │   │       │   │   }
│   │   │   │       │   }, [isReadOnly])
│   │   │   │       └── return <div ref={terminalRef} className="xterm-container" />
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 PodSelector.tsx
│   │   │   │   └── export const PodSelector: React.FC = () => {
│   │   │   │       ├── const pods = useSWR('/api/k8s/pods', fetchPods)
│   │   │   │       ├── const [selectedPod, setSelectedPod] = useState<string>('')
│   │   │   │       ├── const { connect } = useTerminal()
│   │   │   │       ├── const handleConnect = () => {
│   │   │   │       │   └── connect(selectedPod)
│   │   │   │       │   }
│   │   │   │       └── return (
│   │   │   │           ├── <div className="pod-selector">
│   │   │   │           │   ├── <select value={selectedPod} onChange={e => setSelectedPod(e.target.value)}>
│   │   │   │           │   │   └── {pods.map(p => <option key={p.name} value={p.name}>{p.name}</option>)}
│   │   │   │           │   ├── </select>
│   │   │   │           │   └── <button onClick={handleConnect}>Connect</button>
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 TerminalTabs.tsx
│   │   │   │   └── export const TerminalTabs: React.FC = () => {
│   │   │   │       ├── const [sessions, setSessions] = useState<Session[]>([])
│   │   │   │       ├── const [activeSession, setActiveSession] = useState<string>('')
│   │   │   │       └── return (
│   │   │   │           ├── <div className="terminal-tabs">
│   │   │   │           │   └── {sessions.map(s => (
│   │   │   │           │       <div 
│   │   │   │           │         key={s.id} 
│   │   │   │           │         className={`tab ${activeSession === s.id ? 'active' : ''}`}
│   │   │   │           │         onClick={() => setActiveSession(s.id)}
│   │   │   │           │       >
│   │   │   │           │         {s.pod}
│   │   │   │           │         <button onClick={() => closeSession(s.id)}>×</button>
│   │   │   │           │       </div>
│   │   │   │           │     ))}
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 ReadOnlyToggle.tsx
│   │   │   │   └── export const ReadOnlyToggle: React.FC = () => {
│   │   │   │       ├── const { isReadOnly, setReadOnly } = useTerminal()
│   │   │   │       └── return (
│   │   │   │           ├── <div className="read-only-toggle">
│   │   │   │           │   ├── <label>
│   │   │   │           │   │   ├── <input 
│   │   │   │           │   │   │   type="checkbox" 
│   │   │   │           │   │   │   checked={isReadOnly}
│   │   │   │           │   │   │   onChange={e => setReadOnly(e.target.checked)}
│   │   │   │           │   │   │ />
│   │   │   │           │   │   └── Read-Only Mode
│   │   │   │           │   ├── </label>
│   │   │   │           │   └── {isReadOnly && <span className="badge">Safe View</span>}
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 SessionInfo.tsx
│   │   │       └── export const SessionInfo: React.FC = () => {
│   │   │           ├── const { sessionId, connectedTime, idleTimeout } = useTerminal()
│   │   │           └── return (
│   │   │               ├── <div className="session-info">
│   │   │               │   ├── <span>Session: {sessionId.slice(0, 8)}</span>
│   │   │               │   ├── <span>Connected: {formatDuration(Date.now() - connectedTime)}</span>
│   │   │               │   └── <span>Idle timeout: {idleTimeout}min</span>
│   │   │               └── </div>
│   │   │           )
│   │   │           }
│   │   │
│   │   ├── 📁 hooks/
│   │   │   ├── 📄 useTerminal.ts
│   │   │   │   ├── import { useState, useCallback, useEffect } from 'react'
│   │   │   │   └── export const useTerminal = () => {
│   │   │   │       ├── const [ws, setWs] = useState<WebSocket | null>(null)
│   │   │   │       ├── const [sessionId, setSessionId] = useState<string>('')
│   │   │   │       ├── const [isReadOnly, setIsReadOnly] = useState(false)
│   │   │   │       ├── const [connectedTime, setConnectedTime] = useState(0)
│   │   │   │       ├── const idleTimeout = 15  // minutes
│   │   │   │       ├── const connect = useCallback((pod: string) => {
│   │   │   │       │   ├── // Get JWT token for WebSocket authentication
│   │   │   │       │   ├── const token = localStorage.getItem('jwt_token')
│   │   │   │       │   ├── const wsUrl = `wss://bff.superbet:8080/ws/terminal?pod=${pod}&token=${token}`
│   │   │   │       │   ├── const websocket = new WebSocket(wsUrl)
│   │   │   │       │   ├── websocket.binaryType = 'arraybuffer'
│   │   │   │       │   ├── websocket.onopen = () => {
│   │   │   │       │   │   ├── setConnectedTime(Date.now())
│   │   │   │       │   │   └── setSessionId(generateSessionId())
│   │   │   │       │   │   }
│   │   │   │       │   ├── websocket.onclose = () => {
│   │   │   │       │   │   └── // Auto-reconnect logic
│   │   │   │       │   │       setTimeout(() => connect(pod), 1000)
│   │   │   │       │   │   }
│   │   │   │       │   └── setWs(websocket)
│   │   │   │       │   }, [])
│   │   │   │       ├── const disconnect = useCallback(() => {
│   │   │   │       │   ├── if (ws) {
│   │   │   │       │   │   ├── ws.close()
│   │   │   │       │   │   └── setWs(null)
│   │   │   │       │   │   }
│   │   │   │       │   }, [ws])
│   │   │   │       ├── const send = useCallback((data: string) => {
│   │   │   │       │   ├── if (ws && ws.readyState === WebSocket.OPEN) {
│   │   │   │       │   │   ├── ws.send(data)
│   │   │   │       │   │   └── // Send to audit (if not read-only)
│   │   │   │       │   │       if (!isReadOnly) {
│   │   │   │       │   │         auditCommand(sessionId, data)
│   │   │   │       │   │       }
│   │   │   │       │   │   }
│   │   │   │       │   }, [ws, isReadOnly, sessionId])
│   │   │   │       ├── const receive = useCallback((callback: (data: ArrayBuffer) => void) => {
│   │   │   │       │   ├── if (ws) {
│   │   │   │       │   │   └── ws.onmessage = (event) => callback(event.data)
│   │   │   │       │   │   }
│   │   │   │       │   }, [ws])
│   │   │   │       └── return { connect, disconnect, send, receive, isReadOnly, setIsReadOnly: setReadOnly, sessionId, connectedTime, idleTimeout }
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 useCommandHistory.ts
│   │   │       └── export const useCommandHistory = () => {
│   │   │           ├── const [history, setHistory] = useState<string[]>([])
│   │   │           ├── const [historyIndex, setHistoryIndex] = useState(-1)
│   │   │           ├── const addCommand = (cmd: string) => {
│   │   │           │   └── setHistory([...history, cmd])
│   │   │           │   }
│   │   │           └── return { history, addCommand, historyIndex, setHistoryIndex }
│   │   │           }
│   │   │
│   │   ├── 📁 utils/
│   │   │   └── 📄 audit.ts
│   │   │       ├── export const auditCommand = async (sessionId: string, command: string) => {
│   │   │       │   ├── // Send to Kafka audit.terminal.commands topic
│   │   │       │   ├── const payload = {
│   │   │       │   │   ├── session_id: sessionId,
│   │   │       │   │   ├── timestamp: Date.now(),
│   │   │       │   │   ├── command,
│   │   │       │   │   ├── user: getCurrentUser(),
│   │   │       │   │   └── redacted: isSensitiveCommand(command)
│   │   │       │   │   }
│   │   │       │   └── await fetch('/api/audit/terminal', { method: 'POST', body: JSON.stringify(payload) })
│   │   │       │   }
│   │   │       ├── const SENSITIVE_COMMANDS = ['kubectl delete', 'kubectl apply', 'kubectl exec', 'rm -rf', 'dd if=']
│   │   │       └── const isSensitiveCommand = (cmd: string) => SENSITIVE_COMMANDS.some(s => cmd.startsWith(s))
│   │   │
│   │   └── 📁 styles/
│   │       └── 📄 terminal.css
│   │           ├── .xterm-container { height: calc(100vh - 150px); }
│   │           ├── .xterm .xterm-viewport { background: #0A0A0F !important; }
│   │           ├── .read-only-toggle .badge { background: var(--accent-warning); padding: 4px 8px; border-radius: 8px; }
│   │           ├── .terminal-tabs { display: flex; gap: 4px; border-bottom: 1px solid #22222E; }
│   │           ├── .terminal-tabs .tab { padding: 8px 16px; cursor: pointer; border-radius: 8px 8px 0 0; }
│   │           ├── .terminal-tabs .tab.active { background: var(--accent-primary); }
│   │           └── .session-info { display: flex; gap: 16px; font-size: 12px; color: #A0A0B0; }
│   │
│   └── 📄 README.md
│       ├── # Terminal Module
│       ├── ## Features
│       ├── - xterm.js + WebSocket interactive shell
│       ├── - Multi-session support (tabs)
│       ├── - Read-Only toggle for safe viewing
│       ├── - mTLS + JWT authentication
│       ├── - 15-min idle session timeout
│       ├── - Kafka audit topic: audit.terminal.commands (180-day TTL)
│       ├── - Sensitive command redaction (kubectl delete/apply/exec)
│       └── ## Security
│           - RBAC-controlled command allowlist
│           - All commands logged to Kafka
│           - Automatic session expiry
│
└── 📁 config/
    ├── 📄 package.json
    ├── 📄 webpack.config.js
    │
    ├── 📁 src/
    │   ├── 📄 index.tsx
    │   │
    │   └── 📁 components/
    │       ├── 📄 ConsulBrowser.tsx
    │       │   └── export const ConsulBrowser: React.FC = () => {
    │       │       ├── const [selectedKey, setSelectedKey] = useState<string>('')
    │       │       ├── const value = useSWR(`/api/consul/kv/${selectedKey}`, fetchConsulKV)
    │       │       └── return (
    │       │           ├── <div className="consul-browser">
    │       │           │   ├── <TreeView keys={consulKeys} onSelect={setSelectedKey} />
    │       │           │   └── <ValueViewer value={value} />
    │       │           └── </div>
    │       │       )
    │       │       }
    │       │
    │       ├── 📄 ConfigDiff.tsx
    │       │   ├── import MonacoEditor from '@monaco-editor/react'
    │       │   └── export const ConfigDiff: React.FC<{ before: string, after: string }> = ({ before, after }) => {
    │       │       └── return (
    │       │           ├── <MonacoEditor
    │       │           │   language="yaml"
    │       │           │   theme="vs-dark"
    │       │           │   original={before}
    │       │           │   modified={after}
    │       │           │   options={{ readOnly: true }}
    │       │           └── />
    │       │       )
    │       │       }
    │       │
    │       └── 📄 VaultSecrets.tsx
    │           └── export const VaultSecrets: React.FC = () => {
    │               ├── const secrets = useSWR('/api/vault/secrets', fetchVaultSecrets)
    │               └── return (
    │                   ├── <Card title="Vault Secrets">
    │                   │   └── <table>
    │                   │       {secrets.map(s => (
    │                   │         <tr key={s.path}>
    │                   │           <td>{s.path}</td>
    │                   │           <td className="masked">{'*'.repeat(s.value.length)}</td>
    │                   │         </tr>
    │                   │       ))}
    │                   │     </table>
    │                   └── </Card>
    │               )
    │               }
    │
    └── 📁 styles/
        └── 📄 config.css
```

---

## 📊 BÖLÜM D İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Modüller** | 4 (k8s, argocd, terminal, config) |
| **Toplam Dosya** | 22+ |
| **React Bileşenleri** | 18+ |
| **xterm.js Addons** | 2 (FitAddon, WebLinksAddon) |
| **KRİTİK** | Terminal ⭐⭐⭐ (xterm.js, WebSocket, Read-Only toggle, Kafka audit) |

---

## 🔗 DEVAMI

**Son Dosya:** `PROJECT_TREE_v3.1_PART6_Frontend_E.md`  
**İçerik:** MLOps + Admin Modülleri + Proto + BFF (Hyperparameters, Experiments, Audit Logs, Go Backend)

---

**Versiyon:** SUPERBET GENESIS v3.1 Frontend Tree - PART 6D  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md
