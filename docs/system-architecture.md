# OpenChamber System Architecture

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Scope:** Desktop, Web, VS Code Extension

---

## High-Level Architecture

OpenChamber uses a **layered architecture** where each deployment target (Desktop/Web/VS Code) loads a shared React UI library connected to different backend systems.

```
┌─────────────────────────────────────────────────────────────┐
│                    UI Layer (@openchamber/ui)               │
│  React 19 + Zustand (36+ stores) + Tailwind CSS 4          │
│  Components: chat, git, terminal, settings, session mgmt    │
└────────────────────────────────────────────────────────────┬┘
                              ▲
                              │
      ┌───────────────────────┼────────────────────────┐
      │                       │                        │
      ▼                       ▼                        ▼
┌──────────────┐      ┌──────────────┐       ┌──────────────┐
│ Web Package  │      │ Desktop App  │       │ VS Code Ext  │
│ (Express)    │      │ (Tauri 2)    │       │ (TypeScript) │
│              │      │              │       │              │
│ 100+ API     │      │ Sidecar      │       │ Webview      │
│ endpoints    │      │ server       │       │ (React)      │
└──────┬───────┘      └──────┬───────┘       └──────┬───────┘
       │                     │                      │
       └─────────────────────┼──────────────────────┘
                             │
                  ┌──────────┴──────────┐
                  │                     │
                  ▼                     ▼
         ┌──────────────────┐   ┌───────────────┐
         │ OpenCode Server  │   │ Local Browser │
         │ (subprocess)     │   │ Runtime APIs  │
         └──────────────────┘   └───────────────┘
```

---

## Runtime Flows by Deployment

### 1. Web (PWA) Runtime

**Initialization Flow:**
```
User visits https://openchamber.example.com
  ↓
Browser downloads:
  - index.html (with script tags)
  - JavaScript chunks (main-*.js, vendor-*.js)
  - CSS chunks (auto-imported)
  - service-worker.js (registered)
  ↓
main.tsx executes:
  1. Create RuntimeAPIs (terminal, git, files, settings, github, push)
  2. Register Service Worker (PWA for push notifications)
  3. Import @openchamber/ui/main and mount React app
  ↓
App loads user settings from localStorage + /api/config/settings
App connects to OpenCode via /api/event (SSE stream)
App renders UI with Zustand stores hydrated from localStorage
```

**Server-Client Data Flow:**
```
┌─────────────────────────────────────────────────────────┐
│ Browser (React App)                                     │
│                                                         │
│  stores/ (Zustand)                                      │
│  ├── sessionStore                                       │
│  ├── messageStore                                       │
│  ├── uiStore (theme, layout)                           │
│  └── ... (36+ stores)                                  │
└─────────────────────────────────────────────────────────┘
                         ▲▼ (HTTP + WebSocket)
┌─────────────────────────────────────────────────────────┐
│ Express Server (packages/web/server/index.js)           │
│                                                         │
│  Routes (100+):                                         │
│  - GET /api/config/settings → localStorage restore     │
│  - POST /api/sessions/:id/message → forward to OpenCode│
│  - GET /api/event → SSE stream (real-time updates)     │
│  - WS /ws/terminal/:id → terminal I/O stream           │
│  - GET /api/git/* → git operations via simple-git      │
│  - GET /api/github/* → GitHub API via @octokit/rest    │
└─────────────────────────────────────────────────────────┘
                         ▲▼
┌─────────────────────────────────────────────────────────┐
│ OpenCode Server (subprocess or external)                │
│                                                         │
│  - Inference endpoints (/api/session/:id/message)      │
│  - Model metadata                                       │
│  - Tool execution                                       │
└─────────────────────────────────────────────────────────┘
```

**Key Endpoints by Category:**

| Category | Example Endpoints |
|----------|-------------------|
| **Session** | POST /api/sessions/:id/message, GET /api/sessions/snapshot |
| **Config** | GET/PUT /api/config/settings, GET /api/config/themes |
| **Git** | GET /api/git/status, POST /api/git/commit, POST /api/git/push |
| **GitHub** | POST /api/github/auth/start, POST /api/github/pr |
| **Terminal** | POST /api/terminal/create, WS /ws/terminal/:sessionId |
| **OpenCode** | GET /api/event (SSE), POST /api/opencode/* (proxy) |
| **Tunnels** | POST /api/openchamber/tunnel/start, GET /api/openchamber/tunnel/status |
| **Push** | POST /api/push/subscribe, GET /api/push/vapid-public-key |

---

### 2. Desktop (macOS) Runtime

**Initialization Flow:**
```
App Launch (main.rs)
  ↓
Tauri shows splash window
  ↓
Spawn sidecar (packages/web/server index.js compiled binary)
  └─ Path: src-tauri/sidecars/openchamber-server-{arch}
  └─ Environment: OPENCHAMBER_HOST=127.0.0.1, OPENCHAMBER_DIST_DIR=resources/web-dist
  └─ Health check: http://127.0.0.1:RANDOM_PORT (5s intervals)
  ↓
Probe remote hosts (if configured for SSH tunnel)
  └─ soft timeout: 5s, hard timeout: 30s
  ↓
Navigate webview to selected URL:
  └─ Priority: env override → config default → local sidecar
  ├─ Dev: http://127.0.0.1:5173 (Vite HMR)
  ├─ Prod: http://127.0.0.1:RANDOM_PORT (sidecar)
  └─ Remote: http://127.0.0.1:FORWARD_PORT (SSH tunnel)
  ↓
Inject init script (theme, local origin header)
  ↓
UI loads from resources/web-dist/ (bundled)
  ↓
React app mounts + stores hydrate from disk (~/.openchamber/settings.json)
```

**Desktop-Specific Data Flow:**
```
┌──────────────────────────────────┐
│ macOS Webview (React App)        │
│                                  │
│ IPC invoke() calls ↔ Rust        │
│ (window control, file ops)       │
└──────────────────────────────────┘
         ▲▼ (HTTP local)
┌──────────────────────────────────┐
│ Sidecar (Express + UI bundles)   │
│                                  │
│ Port: RANDOM (10000-60000)       │
│ Binding: 127.0.0.1 (secure)      │
└──────────────────────────────────┘
         ▲▼ (HTTP/WebSocket)
┌──────────────────────────────────┐
│ Embedded OpenCode Server         │
│ (spawned subprocess)             │
│                                  │
│ Port: RANDOM                     │
└──────────────────────────────────┘
```

**IPC Commands (Tauri invoke):**
- `desktop_new_window()` → Create new window
- `desktop_open_path(path)` → Open file in Finder
- `desktop_set_window_theme(theme)` → Apply theme
- `desktop_ssh_connect(instance_id)` → Establish SSH tunnel
- `desktop_notify(payload)` → Show native notification

---

### 3. VS Code Extension Runtime

**Initialization Flow:**
```
VS Code loads extension (extension.ts)
  ↓
Extension activates:
  1. Register commands (chat, session, git, etc.)
  2. Register tree views (sessions, files)
  3. Check for running OpenCode server (health check)
  ↓
If no server running:
  └─ Spawn local web server (packages/web/server/index.js)
  └─ Port: 3001 (configurable)
  ├─ Environment: OPENCODE_PORT=4096 (or spawn OpenCode)
  └─ Health check: http://127.0.0.1:3001/health
  ↓
Create webview panel:
  └─ HTML: Vite-built React SPA
  └─ Script: bridge.ts (RPC protocol)
  └─ CSS: From @openchamber/ui
  ↓
Webview loads React app from localhost:3001
  ↓
Bridge protocol establishes communication:
  └─ Webview → Extension: postMessage(api:*, command, themeChange)
  └─ Extension → Webview: window.postMessage()
```

**Extension-Webview Bridge Protocol:**

```typescript
// Messages from Webview → Extension
type WebviewMessage =
  | { type: 'api:proxy'; method: string; url: string; data?: any }
  | { type: 'api:session:message'; sessionId: string; message: any }
  | { type: 'api:sse:subscribe'; topic: string }
  | { type: 'api:config/*'; action: string }
  | { type: 'api:vscode/*'; action: string }
  | { type: 'command'; command: string; args?: any[] }
  | { type: 'themeChange'; theme: 'light' | 'dark' };

// Messages from Extension → Webview
type ExtensionMessage =
  | { type: 'response'; requestId: string; data: any }
  | { type: 'error'; requestId: string; error: string }
  | { type: 'sse:message'; topic: string; data: any }
  | { type: 'theme:change'; theme: 'light' | 'dark' };
```

---

## State Management Architecture

### Zustand Store Hierarchy

**36+ Stores organized by domain:**

```
Core Session Management:
├── useSessionStore        # Active sessions, current session
├── useMessageStore        # Messages, message draft
├── useWorkspaceStore      # Workspace paths, project config

UI State:
├── useUIStore             # Theme, layout, sidebar collapsed
├── useViewStore           # Current view (chat/git/terminal)
├── useSettingsStore       # User settings (shortcuts, font size)

Git & GitHub:
├── useGitStore            # Git status, staged files, branches
├── useGitHubStore         # GitHub auth state, repos, PRs
├── useWorktreeStore       # Worktree list, active worktree

Features:
├── useVoiceStore          # Voice input/output state
├── useTerminalStore       # Terminal sessions, active session
├── useMultiRunStore       # Agent runs, worktrees for comparison
├── usePlanStore           # Plan/build mode state
└── ... (more feature stores)
```

**Key Zustand Patterns:**
```typescript
// Example: useSessionStore
interface SessionState {
  sessions: Session[];
  activeSessionId: string | null;
  addSession: (session: Session) => void;
  setActive: (id: string) => void;
  updateMessage: (id: string, message: Message) => void;
}

export const useSessionStore = create<SessionState>()(
  devtools(
    persist(
      (set) => ({
        sessions: [],
        activeSessionId: null,
        addSession: (session) => set((state) => ({
          sessions: [...state.sessions, session]
        })),
        // ...
      }),
      { name: 'session-store' }
    )
  )
);

// Usage in component:
export function ChatView() {
  const { sessions, activeSessionId } = useSessionStore();
  // Component renders based on store state
}
```

**Persistence Strategy:**
- Stores with `persist` middleware save to localStorage (web) or disk (desktop)
- Settings restored on app launch
- Debounced saves (500ms) for performance

---

## OpenCode Integration

### Connection Models

**Model 1: Auto-Spawned (Web/Desktop)**
```
User runs: openchamber serve
  ↓
Server startup:
  1. Check OPENCODE_PORT env var
  2. If not set: Spawn OpenCode subprocess
     └─ Command: opencode serve --port 4095
     └─ Environment: inherit PATH, SSH_AUTH_SOCK
  3. Wait for health check (GET http://127.0.0.1:4095/health)
  4. Store port in memory + cache file
```

**Model 2: External (Docker, VPN, Systemd)**
```
User sets: OPENCODE_PORT=4095 OPENCODE_SKIP_START=true openchamber
  ↓
Server startup:
  1. Skip OpenCode spawn
  2. Connect to external server at OPENCODE_HOST (default localhost:4095)
  3. Proxies all /api/opencode/* requests directly
```

**Model 3: Custom Hostname (Managed OpenCode)**
```
Docker env: OPENCHAMBER_OPENCODE_HOSTNAME=0.0.0.0
  ↓
Server startup:
  1. Spawn OpenCode with --hostname 0.0.0.0 (not default 127.0.0.1)
  2. Accessible from container network
  3. Web server also listens on --host 0.0.0.0
```

### Message Streaming Architecture

**Part-Based Message Flow:**
```
User sends: "Explain this code"
  ↓
HTTP POST /api/sessions/:id/message
  ↓
Server proxies to OpenCode WebSocket
  ↓
OpenCode processes + streams response with parts:
  [TEXT part] "Here's the explanation..."
  [TOOL part] { type: "exec", command: "npm test" }
  [TEXT part] "Running tests..."
  [TOOL_OUTPUT part] { stdout: "..." }
  ↓
Server transforms parts → Message model
  ↓
Server sends via SSE /api/event stream:
  data: { sessionId, messageId, part: { type, content } }
  ↓
Frontend (useMessageStore) accumulates parts:
  messageStore.addPart(messageId, part)
  ↓
UI re-renders message with all parts
  └─ Text renders via MarkdownRenderer
  └─ Tools render via TooComponentRenderer (diff, permission, etc.)
  └─ Tool output renders via OutputRenderer
```

---

## Terminal Architecture

### PTY Over WebSocket

```
┌──────────────────┐
│ Browser Terminal │
│   (Ghostty Web)  │
└─────────┬────────┘
          │
          │ WS messages (input, resize)
          ▼
┌──────────────────────────────┐
│ Node PTY Handler             │
│ (packages/web/server/lib)    │
│                              │
│ 1. Create PTY session        │
│ 2. Fork shell (/bin/bash)    │
│ 3. Listen to stdout/stderr   │
│ 4. Stream to browser via WS  │
└──────────────────────────────┘
          ▲▼
┌──────────────────────────────┐
│ Shell Process                │
│ (user's shell + CWD)         │
│                              │
│ - Inherits PATH, SSH_AUTH    │
│ - Can execute git, npm, etc. │
└──────────────────────────────┘
```

**API Flow:**
```
1. POST /api/terminal/create
   ├─ Request: { cwd: '/path/to/project', shell: '/bin/bash' }
   └─ Response: { sessionId, pid, initialMessage: 'welcome' }

2. WS /ws/terminal/:sessionId
   ├─ Client → Server: { type: 'input', data: 'ls -la\n' }
   ├─ Server → Client: { type: 'output', data: 'total 42\n...' }
   └─ Client → Server: { type: 'resize', cols: 120, rows: 40 }

3. POST /api/terminal/:sessionId/input
   └─ Send keyboard input without WebSocket
```

**Key Implementation:**
- `node-pty` for macOS/Linux
- `bun-pty` for Bun runtime (fallback)
- Handles heavy output (1000+ lines/sec) gracefully
- Per-session state tracking
- CWD isolation per session

---

## Git & GitHub Integration

### Git Operations Flow

```
User clicks "Commit" in Git sidebar
  ↓
Frontend: POST /api/git/commit
  ├─ Payload: { message, files: ['src/index.ts'] }
  ↓
Server (lib/git/):
  1. Initialize simple-git client for workspace
  2. Stage files: await git.add(files)
  3. Create commit: await git.commit(message)
  4. Update branch refs
  ↓
Server response: { commit: { hash, message, author } }
  ↓
Frontend updates:
  1. useGitStore.updateStatus() → fetch fresh status
  2. MessageStore adds TOOL_OUTPUT part with commit hash
  ↓
UI reflects: Git sidebar shows new commit
```

**GitHub OAuth Device Flow:**
```
User clicks "Authenticate with GitHub"
  ↓
GET /api/github/auth/start
  ├─ Server calls GitHub device_authorization endpoint
  ├─ Returns: { device_code, user_code, verification_uri }
  └─ Response: { userCode: 'XXXX-YYYY', verificationUri: '...' }
  ↓
Frontend displays: "Visit github.com/login/device, enter XXXX-YYYY"
  └─ User opens browser, enters code, approves
  ↓
Frontend polls: POST /api/github/auth/complete
  ├─ Server polls GitHub access_token endpoint
  ├─ Once approved, stores token in ~/.opencode/auth.json (mode 600)
  └─ Response: { success: true, email: 'user@example.com' }
  ↓
useGitHubStore.setAuthenticated(true)
  ↓
GitHub features unlocked: PR creation, issue links, repo listing
```

---

## Tunnel Architecture (Cloudflare)

### Three Tunnel Modes

**Mode 1: Quick (Temporary, No Auth)**
```
User clicks "Start Tunnel" → mode: quick
  ↓
POST /api/openchamber/tunnel/start
  ├─ Spawn cloudflared process: cloudflared tunnel --url http://127.0.0.1:3000
  ├─ Parse output for tunnel URL: https://abc-def-ghi.trycloudflare.com
  └─ Store in server memory
  ↓
Response: { url, qrCode, connectLink }
  ↓
Frontend shows:
  └─ QR code for easy mobile scanning
  └─ Copy-paste link
  └─ Password protection: UI_PASSWORD env var
  ↓
Anyone with URL can connect (one-time URL expires on stop)
```

**Mode 2: Managed-Remote (Custom Domain)**
```
User adds tunnel profile:
  openchamber tunnel profile add \
    --provider cloudflare \
    --mode managed-remote \
    --hostname app.example.com \
    --token <CF_TOKEN>
  ↓
POST /api/openchamber/tunnel/start --profile prod-main
  ├─ Load profile from ~/.config/openchamber/tunnel-cli-state.json
  ├─ Spawn: cloudflared tunnel --hostname app.example.com --token <TOKEN>
  ├─ Validate: health check GET app.example.com
  └─ Store tunnel state
  ↓
Access: https://app.example.com (always available if running)
```

**Mode 3: Managed-Local (Corporate Tunnel)**
```
User provides cloudflared config: ~/.cloudflared/config.yml
  ├─ Specifies: tunnel.id, credentials.json path, ingress rules
  ↓
POST /api/openchamber/tunnel/start --mode managed-local --config /path
  ├─ Spawn: cloudflared tunnel run myapp (reads config.yml)
  ├─ Tunnel persists on Cloudflare account
  └─ IP restrictions, auth rules applied
  ↓
Access: https://myapp.tunnels.example.com (or custom domain)
```

**Tunnel State Management:**
```
~/.config/openchamber/tunnel-cli-state.json
{
  "instances": {
    "3000": {
      "mode": "quick",
      "pid": 12345,
      "url": "https://abc-def-ghi.trycloudflare.com",
      "startedAt": "2026-03-24T...",
      "connectLinks": ["https://.../connect/token123"]
    }
  }
}
```

---

## Desktop SSH Remote Integration

### SSH Tunnel to Remote OpenChamber

```
User adds SSH instance:
  desktop_ssh_instances_set({
    id: 'prod-main',
    host: 'dev.example.com',
    port: 22,
    user: 'developer',
    keyPath: '~/.ssh/id_ed25519'
  })
  ↓
User clicks "Connect to prod-main"
  ↓
desktop_ssh_connect('prod-main') (Rust, remote_ssh.rs)
  1. Open SSH channel: ssh://developer@dev.example.com:22
  2. Execute on remote: "cd /workspace && openchamber serve --port 4000"
  3. Establish local port forward: 127.0.0.1:LOCAL_PORT → remote:4000
  4. Health check via local port
  5. Store tunnel state in Tauri state
  ↓
Frontend navigates to: http://127.0.0.1:LOCAL_PORT
  ↓
User can:
  └─ Use remote OpenChamber instance
  └─ Edit code on remote machine
  └─ Run remote terminal commands
  ↓
desktop_ssh_disconnect('prod-main')
  ├─ Terminate SSH tunnel
  ├─ Kill remote openchamber process
  └─ Cleanup port forward
```

**Log Streaming:**
```
desktop_ssh_logs('prod-main')
  ├─ Returns: last 1200 lines from OpenCode server stdout
  └─ UI shows real-time logs during connection

desktop_ssh_logs_clear('prod-main')
  └─ Clears log buffer for instance
```

---

## Theming System

### Theme Structure

**Flexoki Theme (Example):**
```
colors:
  primary:
    base: #EC8B49
    hover: #DA702C
    active: #F9AE77
  surface:
    background: #100F0F
    foreground: #CECDC3
    elevated: #1C1A1990
  interactive:
    border: #343331
    focus: #EC8B49
  status:
    success: #A0AF54
    error: #D14D41
    warning: #DA702C

typography:
  fontFamily: "'IBM Plex Mono', monospace"
  fontSize: { sm: '0.875rem', base: '1rem', lg: '1.125rem' }
  lineHeight: { tight: 1.2, normal: 1.5, loose: 1.8 }

spacing:
  xs: '0.25rem'
  sm: '0.5rem'
  md: '1rem'
  lg: '1.5rem'
  xl: '2rem'
```

**Theme Loading:**
1. **Built-in Themes** → Embedded in app (4 variants)
2. **Custom Themes** → ~/.config/openchamber/themes/*.json
3. **Hot Reload** → Settings → Theme → Reload themes (no restart)
4. **Persistence** → Selected theme ID saved to localStorage/settings.json

**CSS Variable Injection:**
```html
<style>
  :root {
    --color-primary: #EC8B49;
    --color-surface-bg: #100F0F;
    --color-text: #CECDC3;
    /* ... all theme colors */
  }
</style>
```

**Component Usage:**
```typescript
// Apply via design-system.css tokens
<div className="bg-[var(--color-surface-bg)] text-[var(--color-text)]">
```

---

## Authentication & Security Flow

### Web UI Authentication

```
User accesses: http://localhost:3000
  ↓
If UI_PASSWORD set:
  ├─ Redirect to: /auth?redirect=/
  ├─ User enters password
  ├─ POST /auth with password
  ├─ Server validates against UI_PASSWORD env var
  ├─ Sets HTTP-only cookie: openchamber-session
  └─ Redirect to original URL
  ↓
If no UI_PASSWORD:
  └─ Direct access to UI
```

### Tunnel Authentication

```
Tunnel connect link: https://tunnel.trycloudflare.com/connect?token=abc123
  ↓
Token is one-time use
  ↓
First visit: Tunnel server validates token
  ├─ Creates session cookie
  └─ Stores in server memory
  ↓
Subsequent visits: Session cookie valid
  ├─ Can connect to http://127.0.0.1:3000
  └─ Optional: UI_PASSWORD still protects UI
  ↓
Token regeneration:
  ├─ New connect link → new token
  ├─ Old token invalidated
  └─ Prevents token leakage
```

### GitHub OAuth (Device Flow)

```
No browser redirect needed
  ↓
1. GET /api/github/auth/start → user_code + verification_uri
2. User scans QR or visits URL manually
3. GitHub shows: "Grant access to OpenChamber?"
4. User approves
5. Frontend polls: POST /api/github/auth/complete
6. Server exchanges device_code for access_token
7. Token stored: ~/.opencode/auth.json (mode 600, not committed)
8. useGitHubStore.setAuthenticated(true)
```

---

## WebSocket & SSE Event Streams

### SSE (/api/event) for OpenCode Updates

```
Frontend: GET /api/event?sessionId=xyz (long-lived connection)
  ↓
Server proxies connection to OpenCode server
  ↓
OpenCode streams events:
  ```
  data: {
    "type": "message_part",
    "sessionId": "xyz",
    "messageId": "msg-123",
    "part": {
      "type": "text",
      "content": "Processing..."
    }
  }

  data: {
    "type": "tool_output",
    "toolId": "exec",
    "output": "..."
  }
  ```
  ↓
Frontend parses SSE data
  ├─ useMessageStore.addPart(messageId, part)
  ├─ useUIStore.setActivityStatus(message)
  └─ UI re-renders in real-time
```

### WebSocket (/ws/terminal/:sessionId) for Terminal I/O

```
Frontend: new WebSocket('ws://127.0.0.1:3000/ws/terminal/term-123')
  ↓
Messages:
  Client → Server: { type: 'input', data: 'npm start\n' }
  Server → Client: { type: 'output', data: 'Starting server...\n' }
  Client → Server: { type: 'resize', cols: 120, rows: 40 }
  ↓
Bidirectional streaming:
  └─ Supports real-time terminal interaction
```

---

## Performance Optimizations

- **Bundle splitting:** Manual Vite chunks — vendor-react, vendor-zustand, vendor-opencode-sdk, vendor-markdown, vendor-radix, vendor-syntax. Target: ~1.2MB total, no single chunk exceeds limit.
- **Lazy loading:** Views loaded on-demand via `React.lazy()` + `Suspense`.
- **Message virtualization:** Only visible messages rendered in chat list.
- **Debounced saves:** Window geometry (500ms), settings (1s).
- **Worker offloading:** Diff computation in Web Worker (`DiffWorkerProvider`).

## Error Handling

- **Client:** `ErrorBoundary` for component crashes, try-catch with toast for API calls, SSE auto-reconnect on disconnect.
- **Server:** Express error middleware returns `{ error, code }`. Route handlers use try-catch → `next(error)`.
- **Desktop:** Tauri Result types, graceful sidecar shutdown on app close.

---

## Summary

OpenChamber's architecture enables:
- **Code Reuse:** Shared UI library across 3 deployment targets
- **Loose Coupling:** Backend agnostic to frontend (Express, Tauri, VS Code APIs)
- **Real-Time:** SSE + WebSocket for responsive UX
- **Offline:** Desktop app functions locally; Web PWA minimal offline
- **Scalability:** Monorepo with clear package boundaries
- **Extensibility:** Plugins via MCP protocol, custom themes, skills catalog

---

**Maintained by:** OpenChamber Team | **License:** MIT
