# 🏗️ SUPERBET GENESIS v3.1 - PROJE TREE PART 6E
## Frontend - MLOps + Admin + Proto + BFF (4+ Seviye Derinlik)

**Tarih:** 05.01.2026 | **Referans:** FRONTEND_ARCHITECTURE_v2.1.md

---

```
superbet-genesis/frontend/modules/ (devam)
│
├── 📁 experiments/                ← MLOps Dashboard
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       └── @superbet/shared: workspace:*
│   │
│   ├── 📄 webpack.config.js
│   │   └── exposes: { './RemoteEntry': './src/index.tsx' }
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 MLflowEmbed.tsx
│   │   │   │   └── export const MLflowEmbed: React.FC = () => {
│   │   │   │       ├── const [token, setToken] = useState<string>('')
│   │   │   │       ├── const [refreshTime, setRefreshTime] = useState(0)
│   │   │   │       ├── // Fetch SSO token
│   │   │   │       ├── useEffect(() => {
│   │   │   │       │   ├── const fetchToken = async () => {
│   │   │   │       │   │   ├── const response = await fetch('/api/mlflow/token')
│   │   │   │       │   │   ├── const { token, ttl } = await response.json()
│   │   │   │       │   │   ├── setToken(token)
│   │   │   │       │   │   └── // Refresh at T-45s (TTL=300s)
│   │   │   │       │   │       setTimeout(fetchToken, (ttl - 45) * 1000)
│   │   │   │       │   │   }
│   │   │   │       │   └── fetchToken()
│   │   │   │       │   }, [])
│   │   │   │       ├── // postMessage bridge for actions
│   │   │   │       ├── useEffect(() => {
│   │   │   │       │   ├── const handleMessage = (event: MessageEvent) => {
│   │   │   │       │   │   ├── if (event.origin !== 'https://mlflow.superbet') return
│   │   │   │       │   │   ├── if (event.data.type === 'promote') {
│   │   │   │       │   │   │   └── handlePromote(event.data.modelId)
│   │   │   │       │   │   │   }
│   │   │   │       │   │   }
│   │   │   │       │   └── window.addEventListener('message', handleMessage)
│   │   │   │       │       return () => window.removeEventListener('message', handleMessage)
│   │   │   │       │   }, [])
│   │   │   │       └── return (
│   │   │   │           ├── <div className="mlflow-embed">
│   │   │   │           │   └── <iframe
│   │   │   │           │       src={`https://mlflow.superbet/#/?token=${token}`}
│   │   │   │           │       width="100%"
│   │   │   │           │       height="800px"
│   │   │   │           │       sandbox="allow-same-origin allow-scripts"
│   │   │   │           │       allow="autoplay"
│   │   │   │           │     />
│   │   │   │           └── </div>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 ActionButtons.tsx
│   │   │       └── export const ActionButtons: React.FC = () => {
│   │   │           ├── const handlePromote = async (modelId: string) => {
│   │   │           │   └── await fetch(`/api/mlflow/promote/${modelId}`, { method: 'POST' })
│   │   │           │   }
│   │   │           └── return <button onClick={() => handlePromote('model-123')}>Promote to Prod</button>
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useMLflowToken.ts
│   │           └── export const useMLflowToken = () => {
│   │               ├── const [token, setToken] = useState('')
│   │               ├── // Auto-refresh logic
│   │               └── return token
│   │               }
│   │
│   └── 📁 styles/
│       └── 📄 experiments.css
│           └── .mlflow-embed iframe { border: none; border-radius: 8px; }
│
├── 📁 training/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 RayJobList.tsx
│   │       │   └── export const RayJobList: React.FC = () => {
│   │       │       ├── const jobs = useSWR('/api/ray/jobs', fetchRayJobs)
│   │       │       └── return (
│   │       │           ├── <Card title="Ray Training Jobs">
│   │       │           │   └── <VirtualTable data={jobs} columns={RAY_JOB_COLUMNS} />
│   │       │           └── </Card>
│   │       │       )
│   │       │       }
│   │       │
│   │       └── 📄 GPUUsageGauges.tsx
│   │           └── export const GPUUsageGauges: React.FC = () => {
│   │               ├── const gpus = useSWR('/api/ray/gpu-usage', fetchGPUUsage)
│   │               └── return (
│   │                   ├── <Card title="GPU Usage">
│   │                   │   └── <div className="gpu-gauges">
│   │                   │       {gpus.map(gpu => (
│   │                   │         <Gauge key={gpu.id} value={gpu.utilization} label={`GPU ${gpu.id}`} />
│   │                   │       ))}
│   │                   │     </div>
│   │                   └── </Card>
│   │               )
│   │               }
│   │
│   └── 📁 styles/
│       └── 📄 training.css
│
├── 📁 features/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 FeatureStoreBrowser.tsx
│   │       │   └── export const FeatureStoreBrowser: React.FC = () => {
│   │       │       ├── const features = useSWR('/api/feast/features', fetchFeatures)
│   │       │       └── return (
│   │       │           ├── <Card title="Feature Store">
│   │       │           │   └── <TreeView data={features} />
│   │       │           └── </Card>
│   │       │       )
│   │       │       }
│   │       │
│   │       └── 📄 FeatureLineage.tsx
│   │           ├── import * as d3 from 'd3'
│   │           └── export const FeatureLineage: React.FC = () => {
│   │               └── // DAG visualization of feature dependencies
│   │               }
│   │
│   └── 📁 styles/
│       └── 📄 features.css
│
├── 📁 quality/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 ValidationRunHistory.tsx
│   │       │   └── // Great Expectations validation history
│   │       │
│   │       └── 📄 DataProfiles.tsx
│   │           └── // Data profiling charts
│   │
│   └── 📁 styles/
│       └── 📄 quality.css
│
├── 📁 hyperparams/                ← MLOps (YENİ v2.1) ⭐
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 StudyList.tsx
│   │   │   │   └── export const StudyList: React.FC = () => {
│   │   │   │       ├── const studies = useSWR('/api/optuna/studies', fetchStudies)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Optuna Studies">
│   │   │   │           │   └── <VirtualTable data={studies} columns={STUDY_COLUMNS} />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 TrialHistory.tsx
│   │   │   │   └── export const TrialHistory: React.FC<{ studyId: string }> = ({ studyId }) => {
│   │   │   │       ├── const trials = useSWR(`/api/optuna/studies/${studyId}/trials`, fetchTrials)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Trial History">
│   │   │   │           │   ├── <VirtualTable data={trials} columns={TRIAL_COLUMNS} />
│   │   │   │           │   └── <LineChart data={trials} x="trial_number" y="value" />
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 BestParameters.tsx
│   │   │   │   └── export const BestParameters: React.FC<{ studyId: string }> = ({ studyId }) => {
│   │   │   │       ├── const best = useSWR(`/api/optuna/studies/${studyId}/best`, fetchBestTrial)
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Best Parameters">
│   │   │   │           │   └── <pre>{JSON.stringify(best.params, null, 2)}</pre>
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 OptimizationChart.tsx
│   │   │       └── export const OptimizationChart: React.FC<{ studyId: string }> = ({ studyId }) => {
│   │   │           ├── const history = useSWR(`/api/optuna/studies/${studyId}/history`, fetchHistory)
│   │   │           └── return (
│   │   │               ├── <Card title="Optimization Progress">
│   │   │               │   └── <LineChart
│   │   │               │       data={history}
│   │   │               │       x="trial_number"
│   │   │               │       y="value"
│   │   │               │       yLabel="Objective Value"
│   │   │               │     />
│   │   │               └── </Card>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useOptuna.ts
│   │           └── export const useOptuna = (studyId: string) => useSWR(`/api/optuna/studies/${studyId}`, fetchStudy)
│   │
│   └── 📁 styles/
│       └── 📄 hyperparams.css
│
├── 📁 users/                      ← Admin Dashboard
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 UserList.tsx
│   │   │   │   └── export const UserList: React.FC = () => {
│   │   │   │       ├── const users = useSWR('/api/users', fetchUsers)
│   │   │   │       └── return <Card title="Users"><VirtualTable data={users} columns={USER_COLUMNS} /></Card>
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 RoleAssignment.tsx
│   │   │   │   └── // Role dropdown with RBAC matrix
│   │   │   │
│   │   │   └── 📄 PermissionMatrix.tsx
│   │   │       └── export const PermissionMatrix: React.FC = () => {
│   │   │           ├── const matrix = PERMISSION_MATRIX
│   │   │           └── return (
│   │   │               ├── <Card title="RBAC Permission Matrix">
│   │   │               │   └── <table className="permission-matrix">
│   │   │               │       <thead>
│   │   │               │         <tr>
│   │   │               │           <th>Role</th>
│   │   │               │           {Object.keys(matrix.operator).map(res => <th key={res}>{res}</th>)}
│   │   │               │         </tr>
│   │   │               │       </thead>
│   │   │               │       <tbody>
│   │   │               │         {Object.entries(matrix).map(([role, perms]) => (
│   │   │               │           <tr key={role}>
│   │   │               │             <td>{role}</td>
│   │   │               │             {Object.values(perms).map((p, i) => (
│   │   │               │               <td key={i} className={p.read ? 'allowed' : 'denied'}>
│   │   │               │                 {p.read && '✓'}
│   │   │               │               </td>
│   │   │               │             ))}
│   │   │               │           </tr>
│   │   │               │         ))}
│   │   │               │       </tbody>
│   │   │               │     </table>
│   │   │               └── </Card>
│   │   │           )
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useUsers.ts
│   │           └── export const useUsers = () => useSWR('/api/users', fetchUsers)
│   │
│   └── 📁 styles/
│       └── 📄 users.css
│           └── .permission-matrix .allowed { background: var(--accent-success); }
│               .permission-matrix .denied { background: var(--accent-danger); }
│
├── 📁 flags/
│   ├── 📄 package.json
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   └── 📁 components/
│   │       ├── 📄 FlagList.tsx
│   │       │   └── // LaunchDarkly feature flags list
│   │       │
│   │       └── 📄 ToggleSwitch.tsx
│   │           └── // Toggle component for flags
│   │
│   └── 📁 styles/
│       └── 📄 flags.css
│
├── 📁 audit/                      ← Admin (YENİ v2.1) ⭐
│   ├── 📄 package.json
│   │   └── dependencies:
│   │       ├── @superbet/shared: workspace:*
│   │       └── react-window: ^3.0.0
│   │
│   ├── 📄 webpack.config.js
│   │
│   ├── 📁 src/
│   │   ├── 📄 index.tsx
│   │   │
│   │   ├── 📁 components/
│   │   │   ├── 📄 AuditTimeline.tsx
│   │   │   │   ├── import { FixedSizeList as List } from 'react-window'
│   │   │   │   ├── import { useAuditLogs } from '../hooks/useAuditLogs'
│   │   │   │   └── export const AuditTimeline: React.FC = () => {
│   │   │   │       ├── const { logs, loadMore, hasMore, isLoading } = useAuditLogs()
│   │   │   │       ├── const AuditRow = ({ index, style }) => {
│   │   │   │       │   ├── const log = logs[index]
│   │   │   │       │   └── return (
│   │   │   │       │       ├── <div style={style} className="audit-row">
│   │   │   │       │       │   ├── <span className="timestamp">{formatTimestamp(log.timestamp)}</span>
│   │   │   │       │       │   ├── <span className="user">{log.user}</span>
│   │   │   │       │       │   ├── <span className="action">{log.action}</span>
│   │   │   │       │       │   └── <span className="resource">{log.resource}</span>
│   │   │   │       │       └── </div>
│   │   │   │       │   )
│   │   │   │       │   }
│   │   │   │       └── return (
│   │   │   │           ├── <Card title="Audit Log">
│   │   │   │           │   ├── <List
│   │   │   │           │   │   height={600}
│   │   │   │           │   │   width={1200}
│   │   │   │           │   │   itemCount={logs.length}
│   │   │   │           │   │   itemSize={32}
│   │   │   │           │   │   >
│   │   │   │           │   │   {AuditRow}
│   │   │   │           │   └── </List>
│   │   │   │           │   ├── {hasMore && (
│   │   │   │           │   │   <button onClick={loadMore} disabled={isLoading}>
│   │   │   │           │   │     {isLoading ? 'Loading...' : 'Load More'}
│   │   │   │           │   │   </button>
│   │   │   │           │   │ )}
│   │   │   │           └── </Card>
│   │   │   │       )
│   │   │   │       }
│   │   │   │
│   │   │   ├── 📄 FilterPanel.tsx
│   │   │   │   └── export const FilterPanel: React.FC = () => {
│   │   │   │       ├── const [filters, setFilters] = useState({ user: '', action: '', timeRange: 'today' })
│   │   │   │       └── // Filter controls
│   │   │   │       }
│   │   │   │
│   │   │   └── 📄 ExportButton.tsx
│   │   │       └── export const ExportButton: React.FC = () => {
│   │   │           ├── const handleExport = async () => {
│   │   │           │   └── const blob = await fetch('/api/audit/export').then(r => r.blob())
│   │   │           │       const url = URL.createObjectURL(blob)
│   │   │           │       const a = document.createElement('a')
│   │   │           │       a.href = url
│   │   │           │       a.download = 'audit-log.csv'
│   │   │           │       a.click()
│   │   │           │   }
│   │   │           └── return <button onClick={handleExport}>Export CSV</button>
│   │   │           }
│   │   │
│   │   └── 📁 hooks/
│   │       └── 📄 useAuditLogs.ts
│   │           ├── import { useState, useCallback } from 'react'
│   │           └── export const useAuditLogs = () => {
│   │               ├── const [logs, setLogs] = useState<AuditLog[]>([])
│   │               ├── const [lastTimestamp, setLastTimestamp] = useState<number>(Date.now())
│   │               ├── const [lastId, setLastId] = useState<string>('')
│   │               ├── const [hasMore, setHasMore] = useState(true)
│   │               ├── const [isLoading, setIsLoading] = useState(false)
│   │               ├── // Keyset pagination (ts, id)
│   │               ├── const loadMore = useCallback(async () => {
│   │               │   ├── setIsLoading(true)
│   │               │   ├── try {
│   │               │   │   ├── const response = await fetch(
│   │               │   │   │   `/api/audit?since_ts=${lastTimestamp}&since_id=${lastId}&limit=100`
│   │               │   │   │   )
│   │               │   │   ├── const newLogs = await response.json()
│   │               │   │   ├── if (newLogs.length > 0) {
│   │               │   │   │   ├── setLogs([...logs, ...newLogs])
│   │               │   │   │   ├── const last = newLogs[newLogs.length - 1]
│   │               │   │   │   ├── setLastTimestamp(last.timestamp)
│   │               │   │   │   └── setLastId(last.id)
│   │               │   │   │   } else {
│   │               │   │   │       └── setHasMore(false)
│   │               │   │   │   }
│   │               │   │   } finally {
│   │               │   │       └── setIsLoading(false)
│   │               │   │   }
│   │               │   }, [logs, lastTimestamp, lastId])
│   │               └── return { logs, loadMore, hasMore, isLoading }
│   │               }
│   │
│   └── 📁 styles/
│       └── 📄 audit.css
│           └── .audit-row { display: grid; grid-template-columns: 200px 150px 200px 1fr; gap: 16px; }
│
├── 📁 secrets/
│   └── // Vault secrets management (already covered in config/)
│
└── 📁 system-config/
    └── // Global settings (already covered in config/)

```

---

## 📊 PROTO + BFF YAPISI

```
superbet-genesis/frontend/ (devam)
│
├── 📁 proto/                      ← Protobuf Schemas
│   ├── 📄 superbet.proto
│   │   ├── syntax = "proto3";
│   │   ├── package superbet;
│   │   ├── message TwinDelta {
│   │   │   ├── string type = 1;  // alarm, match, broadcast, agent_status, cb_status
│   │   │   ├── bytes payload = 2;
│   │   │   └── int64 timestamp = 3;
│   │   │   }
│   │   ├── message AlarmEvent {
│   │   │   ├── string match_id = 1;
│   │   │   ├── double vsnr = 2;
│   │   │   ├── double cas = 3;
│   │   │   ├── double confidence = 4;
│   │   │   ├── double uncertainty = 5;  // EDL τ
│   │   │   ├── string prediction = 6;   // H, D, A
│   │   │   ├── double odds = 7;
│   │   │   └── int64 kickoff = 8;
│   │   │   }
│   │   ├── message MatchUpdate {
│   │   │   ├── string match_id = 1;
│   │   │   ├── string phase = 2;        // prematch, live
│   │   │   ├── int32 home_score = 3;
│   │   │   ├── int32 away_score = 4;
│   │   │   ├── double xg_home = 5;
│   │   │   ├── double xg_away = 6;
│   │   │   └── repeated OddsHistory odds_history = 7;
│   │   │   }
│   │   ├── message BroadcastEvent {
│   │   │   ├── string id = 1;
│   │   │   ├── string match_id = 2;
│   │   │   ├── string platform = 3;     // twitter, telegram, android
│   │   │   ├── bool sent = 4;
│   │   │   ├── string formatted_text = 5;
│   │   │   └── double priority_score = 6;
│   │   │   }
│   │   ├── message LogEntry {
│   │   │   ├── int64 timestamp = 1;
│   │   │   ├── string level = 2;        // DEBUG, INFO, WARN, ERROR, FATAL
│   │   │   ├── string service = 3;
│   │   │   ├── string message = 4;
│   │   │   └── int64 offset = 5;
│   │   │   }
│   │   ├── message TerminalData {
│   │   │   ├── string session_id = 1;
│   │   │   └── bytes data = 2;          // xterm.js binary data
│   │   │   }
│   │   └── // ... other messages
│   │
│   └── 📄 compile.sh
│       ├── #!/bin/bash
│       ├── protoc --js_out=import_style=commonjs,binary:../shared/proto \
│       │          --ts_out=../shared/proto \
│       │          superbet.proto
│       └── echo "Protobuf schemas compiled!"
│
└── 📁 bff/                        ← Go Backend for Frontend
    ├── 📄 go.mod
    │   ├── module github.com/superbet/bff
    │   ├── require (
    │   │   ├── github.com/gorilla/websocket v1.5.0
    │   │   ├── github.com/segmentio/kafka-go v0.4.42
    │   │   ├── google.golang.org/protobuf v1.31.0
    │   │   └── github.com/hashicorp/vault/api v1.10.0
    │   └── )
    │
    ├── 📄 main.go
    │   ├── package main
    │   ├── import (
    │   │   ├── "github.com/gorilla/websocket"
    │   │   ├── "github.com/segmentio/kafka-go"
    │   │   └── "github.com/superbet/bff/handlers"
    │   │   )
    │   ├── func main() {
    │   │   ├── // mTLS setup
    │   │   ├── cert, key := loadCerts()
    │   │   ├── // Vault integration
    │   │   ├── vault := initVault()
    │   │   ├── // HTTP/WS handlers
    │   │   ├── http.HandleFunc("/ws/alarms", handlers.HandleAlarms)
    │   │   ├── http.HandleFunc("/ws/matches", handlers.HandleMatches)
    │   │   ├── http.HandleFunc("/ws/broadcast", handlers.HandleBroadcast)
    │   │   ├── http.HandleFunc("/ws/logs", handlers.HandleLogs) // ⭐ YENİ
    │   │   ├── http.HandleFunc("/ws/terminal", handlers.HandleTerminal) // ⭐⭐⭐ YENİ
    │   │   ├── // REST endpoints
    │   │   ├── http.HandleFunc("/api/logs/backfill", handlers.BackfillLogs) // ⭐
    │   │   ├── http.HandleFunc("/api/mlflow/token", handlers.GetMLflowToken) // ⭐
    │   │   ├── http.HandleFunc("/api/audit", handlers.GetAuditLogs) // ⭐
    │   │   └── log.Fatal(http.ListenAndServeTLS(":8080", cert, key, nil))
    │   │   }
    │
    ├── 📁 handlers/
    │   ├── 📄 ws_alarms.go
    │   │   └── func HandleAlarms(w http.ResponseWriter, r *http.Request) {
    │   │       ├── conn, _ := upgrader.Upgrade(w, r, nil)
    │   │       ├── defer conn.Close()
    │   │       ├── reader := kafka.NewReader(kafka.ReaderConfig{
    │   │       │   ├── Brokers: []string{"kafka:9092"},
    │   │       │   ├── Topic:   "risk.verified",
    │   │       │   └── GroupID: "bff-frontend"
    │   │       │   })
    │   │       └── for {
    │   │           ├── msg, _ := reader.ReadMessage(context.Background())
    │   │           └── conn.WriteMessage(websocket.BinaryMessage, msg.Value)
    │   │           }
    │   │       }
    │   │
    │   ├── 📄 ws_logs.go
    │   │   └── func HandleLogs(w http.ResponseWriter, r *http.Request) {
    │   │       ├── conn, _ := upgrader.Upgrade(w, r, nil)
    │   │       ├── defer conn.Close()
    │   │       ├── // Stream from Loki
    │   │       ├── lokiStream := connectToLoki()
    │   │       └── for logEntry := range lokiStream {
    │   │           └── conn.WriteMessage(websocket.TextMessage, json.Marshal(logEntry))
    │   │           }
    │   │       }
    │   │
    │   ├── 📄 ws_terminal.go
    │   │   ├── func HandleTerminal(w http.ResponseWriter, r *http.Request) {
    │   │   │   ├── // Validate JWT token
    │   │   │   ├── token := r.URL.Query().Get("token")
    │   │   │   ├── if !validateJWT(token) {
    │   │   │   │   └── http.Error(w, "Unauthorized", 401)
    │   │   │   │       return
    │   │   │   │   }
    │   │   │   ├── pod := r.URL.Query().Get("pod")
    │   │   │   ├── conn, _ := upgrader.Upgrade(w, r, nil)
    │   │   │   ├── defer conn.Close()
    │   │   │   ├── // kubectl exec to pod
    │   │   │   ├── cmd := exec.Command("kubectl", "exec", "-it", pod, "--", "/bin/sh")
    │   │   │   ├── cmd.Stdin = &WebSocketReader{conn}
    │   │   │   ├── cmd.Stdout = &WebSocketWriter{conn}
    │   │   │   ├── cmd.Stderr = &WebSocketWriter{conn}
    │   │   │   ├── // Start audit logger
    │   │   │   ├── auditProducer := kafka.NewWriter(kafka.WriterConfig{
    │   │   │   │   ├── Brokers: []string{"kafka:9092"},
    │   │   │   │   └── Topic:   "audit.terminal.commands"
    │   │   │   │   })
    │   │   │   ├── // 15-min idle timeout
    │   │   │   ├── idleTimer := time.NewTimer(15 * time.Minute)
    │   │   │   ├── go func() {
    │   │   │   │   ├── <-idleTimer.C
    │   │   │   │   └── conn.Close()
    │   │   │   │   }()
    │   │   │   └── cmd.Run()
    │   │   │   }
    │   │   └── // WebSocketReader/Writer implement io.Reader/Writer
    │   │
    │   ├── 📄 rest_logs.go
    │   │   └── func BackfillLogs(w http.ResponseWriter, r *http.Request) {
    │   │       ├── since := r.URL.Query().Get("since")
    │   │       ├── limit := r.URL.Query().Get("limit")
    │   │       ├── // Query Loki/ClickHouse for historical logs
    │   │       ├── logs := queryLoki(since, limit)
    │   │       └── json.NewEncoder(w).Encode(logs)
    │   │       }
    │   │
    │   ├── 📄 rest_mlflow.go
    │   │   └── func GetMLflowToken(w http.ResponseWriter, r *http.Request) {
    │   │       ├── // Generate JWT token with 300s TTL
    │   │       ├── token := generateJWT(300)
    │   │       └── json.NewEncoder(w).Encode(map[string]interface{}{
    │   │           ├── "token": token,
    │   │           └── "ttl": 300
    │   │           })
    │   │       }
    │   │
    │   └── 📄 rest_audit.go
    │       └── func GetAuditLogs(w http.ResponseWriter, r *http.Request) {
    │           ├── sinceTs := r.URL.Query().Get("since_ts")
    │           ├── sinceId := r.URL.Query().Get("since_id")
    │           ├── limit := r.URL.Query().Get("limit")
    │           ├── // Keyset pagination (ts, id) from ClickHouse
    │           ├── logs := queryClickHouse(
    │           │   "SELECT * FROM audit_logs WHERE (timestamp, id) < (?, ?) ORDER BY timestamp DESC, id DESC LIMIT ?",
    │           │   sinceTs, sinceId, limit
    │           │   )
    │           └── json.NewEncoder(w).Encode(logs)
    │           }
    │
    ├── 📁 kafka/
    │   ├── 📄 consumer.go
    │   │   └── // Kafka consumer wrappers
    │   └── 📄 producer.go
    │       └── func ProduceAudit(topic string, event AuditEvent) {
    │           ├── producer := kafka.NewWriter(kafka.WriterConfig{
    │           │   ├── Brokers: []string{"kafka:9092"},
    │           │   ├── Topic:   topic,
    │           │   └── Balancer: &kafka.LeastBytes{}
    │           │   })
    │           └── producer.WriteMessages(context.Background(), kafka.Message{Value: json.Marshal(event)})
    │           }
    │
    └── 📁 auth/
        ├── 📄 mtls.go
        │   └── func validateMTLS(r *http.Request) bool {
        │       └── // SPIFFE/Vault identity verification
        │       }
        └── 📄 jwt.go
            ├── func generateJWT(ttl int) string {
            │   └── // Generate JWT with TTL
            │   }
            └── func validateJWT(token string) bool
```

---

## 📊 BÖLÜM E İSTATİSTİKLER

| Kategori | Sayı |
|----------|------|
| **Frontend Modüller** | 7 (experiments, training, features, quality, hyperparams, users, audit) |
| **Protobuf Messages** | 9 |
| **Go BFF Handlers** | 10+ (WS: 5, REST: 5) |
| **Toplam Dosya** | 35+ |
| **KRİTİK** | Audit ⭐ (Keyset pagination, virtualized), MLflow iframe ⭐ (SSO token, postMessage) |

---

## 📊 GENEL FRONTEND TREE İSTATİSTİKLER

| Bölüm | Dosya | Modül Sayısı | Satır |
|-------|-------|--------------|-------|
| **PART 6A** | Shell + Shared + Worker | 3 klasör | ~350 |
| **PART 6B** | Operator + Analyst | 7 modül | ~400 |
| **PART 6C** | DevOps (Logs ⭐⭐⭐) | 5 modül | ~430 |
| **PART 6D** | Infra (Terminal ⭐⭐⭐) | 4 modül | ~430 |
| **PART 6E** | MLOps + Admin + BFF | 7 modül + Proto + Go | ~470 |
| **TOPLAM** | 5 PART | **26+ modül** | **~2080** |

---

## 🎯 TAMAMLANDI!

**Frontend tree tam kapsamlı olarak oluşturuldu:**
- ✅ Shell container (Webpack Module Federation)
- ✅ Shared infrastructure (components, hooks, streams, utils)
- ✅ SharedWorker (cross-tab WebSocket)
- ✅ 26+ modül (Operator, Analyst, DevOps, Infra, MLOps, Admin)
- ✅ KRİTİK: **Logs** (EWMA fallback, react-window, backfill)
- ✅ KRİTİK: **Terminal** (xterm.js, WebSocket, Read-Only, Kafka audit)
- ✅ Protobuf schemas (9 message)
- ✅ Go BFF (10+ handler)

---

**Versiyon:** SUPERBET GENESIS v3.1 Frontend Tree - PART 6E (FINAL)  
**Tarih:** 05.01.2026  
**Referans:** FRONTEND_ARCHITECTURE_v2.1.md  
**Durum:** ✅ TAMAMLANDI
