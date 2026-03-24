# OpenChamber Codebase Summary

**Generated:** 2026-03-24 | **Monorepo Type:** Bun workspaces | **Total Files:** 1,910 | **Total Tokens:** ~3.2M

---

## Overview

OpenChamber is a **monorepo with 4 packages** (ui, web, desktop, vscode) + root configuration. All share a unified type system and React UI library.

**Repomix Stats:**
- Total source files: 1,910
- Total tokens: 3.2M (3.2B characters)
- Top 5 largest files: sprite.svg (577K tokens), server/index.js (114K), cli.js (40K), bridge.ts (32K), ChatInput.tsx (28K)

---

## Package Structure

### 1. packages/ui (Shared React Library)
**Purpose:** Core UI components, state management, views, and layout shared by web, desktop, and VS Code extension.

**Key Stats:**
- **506 source files** (TypeScript + React)
- **36+ Zustand stores** for global state
- **50+ custom components** + Radix UI integration
- **4 built-in themes** (Flexoki Light/Dark, Fields of the Shire)

**Directory Organization:**
```
components/
├── ui/              # Base UI (button, dialog, select, tooltip, 49 files)
├── layout/          # Main layout (MainLayout, Sidebar, RightSidebar, 13 files)
├── chat/            # Chat interface (MessageList, ChatInput, 34 files)
├── views/           # Views (ChatView, GitView, DiffView, TerminalView, 10 files)
├── session/         # Session management (SessionSidebar, dialogs, 20 files)
├── sections/        # Settings panels (agents, git, commands, mcp, 14+ dirs)
├── terminal/        # Terminal UI (minimal, via Ghostty Web)
├── voice/           # Voice I/O
├── multirun/        # Multi-agent orchestration
├── comments/        # Inline comments on diffs/files
├── desktop/         # Desktop-specific UI (menus, SSH)
├── icons/           # Icon components
├── auth/            # Auth UI
├── onboarding/      # Setup flow
├── mcp/             # MCP protocol UI
└── providers/       # Context providers

contexts/            # React contexts (11 files)
hooks/               # Custom hooks (42 files)
stores/              # Zustand stores (36+ files)
lib/                 # Utilities (message handling, SSE, types)
styles/              # Global styles + design tokens (design-system.css)
```

**Entry Point:** src/main.tsx → imports @openchamber/ui/main (consumed by web & extension)

**Key Dependencies:**
- React 19.1.1, TypeScript 5.8.3
- Zustand 5.0.8 (state management)
- Tailwind CSS 4.0 (styling via design tokens)
- Radix UI (accessible primitives)
- CodeMirror 6 (code editor)
- Ghostty Web 0.3.0 (terminal emulator)
- Pierre Diffs 1.1.0-beta.13 (diff rendering)
- Motion 12.23.24 (animations)
- dnd-kit (drag-and-drop)

---

### 2. packages/web (Web Server + PWA)
**Purpose:** Express-based backend serving React UI, proxying OpenCode, managing tunnels, terminals, git/github.

**Key Stats:**
- **14,413-line server/index.js** (100+ REST/WebSocket endpoints)
- **React SPA frontend** (Vite build from @openchamber/ui)
- **CLI:** openchamber command (serve, tunnel, logs, update, stop)
- **Supports:** Node.js 20+ or Bun runtime

**Directory Structure:**
```
bin/
├── cli.js                # CLI entry point + command routing
└── cli-*.js             # CLI support modules

src/                      # Frontend (React, TypeScript)
├── main.tsx             # App entry, PWA registration
├── sw.ts                # Service Worker (push notifications)
└── api/                 # Runtime API implementations
    ├── terminal.ts      # Terminal WebSocket wrapper
    ├── git.ts           # Git operations (HTTP proxy)
    ├── github.ts        # GitHub OAuth + API
    ├── files.ts         # File system listing/search
    ├── settings.ts      # Config persistence
    ├── permissions.ts   # Permission checks
    ├── notifications.ts # Web Notification API + Tauri fallback
    ├── push.ts          # Web Push API
    └── tools.ts         # Tool execution

server/
├── index.js             # Main Express server (14.4K lines)
├── index.d.ts           # TypeScript definitions
└── lib/
    ├── opencode/        # OpenCode subprocess management
    ├── tunnels/         # Cloudflare tunnel handling
    ├── terminal/        # Node PTY + bun-pty support
    ├── git/             # Git utilities + simple-git wrapper
    ├── github/          # GitHub OAuth device flow + API
    ├── quota/           # Model quota tracking (20+ providers)
    ├── skills-catalog/  # Skills discovery & management
    ├── tts/             # Text-to-speech service
    ├── notifications/   # Push notification management
    └── ...              # Other integrations

public/                   # Static assets (icons, manifest)
vite.config.ts           # Vite build config (React, PWA plugin)
tsconfig.json            # TypeScript strict config
```

**API Endpoints (100+):**
- **System:** /health, /api/system/info, /api/system/shutdown
- **Config:** /api/config/settings, /api/config/themes
- **Sessions:** /api/sessions/:id/message, /api/sessions/snapshot
- **Git:** /api/git/status, /api/git/commit, /api/git/push
- **GitHub:** /api/github/auth, /api/github/pr, /api/github/repos
- **Terminal:** /api/terminal/create, /ws/terminal/:sessionId
- **OpenCode:** /api/event (SSE), /api/session/:sessionId/message
- **Tunnels:** /api/openchamber/tunnel/start, /api/openchamber/tunnel/stop
- **Push:** /api/push/vapid-public-key, /api/push/subscribe

**Key Dependencies:**
- Express 5.1.0, TypeScript 5.8.3
- @openchamber/ui (shared React library)
- ws 8.18.3 (WebSocket support)
- node-pty 1.2.0-beta.12 (terminal emulation)
- simple-git 3.28.0 (Git operations)
- @octokit/rest 22.0.1 (GitHub API)
- web-push 3.6.7 (notifications)
- Vite 7.1.2 + PWA plugin (build)

**Runtime Startup:**
```
bin/cli.js serve --port 3000 --host 127.0.0.1 --ui-password secret
  → Express app initialization
  → Spawn or connect to OpenCode subprocess
  → Register 100+ routes
  → Setup WebSocket for terminal/notifications
  → Listen on port (default 3000)
```

---

### 3. packages/desktop (macOS Native App)
**Purpose:** Native macOS desktop application with sidecar OpenCode server, SSH remotes, multi-window support.

**Key Stats:**
- **Tauri 2.9.4** framework (Rust 2021 edition)
- **~3,400 lines** main.rs (window, menu, IPC)
- **~2,970 lines** remote_ssh.rs (SSH tunnel management)
- **23 IPC commands** for native integration
- **Minimum macOS:** 13.0 (Big Sur); Architectures: arm64 + x86_64

**Directory Structure:**
```
src-tauri/
├── src/
│   ├── main.rs                  # Window mgmt, menu, IPC handler, app lifecycle
│   └── remote_ssh.rs            # SSH tunnel, port forward, remote instance mgmt
├── Cargo.toml                   # Rust dependencies
├── tauri.conf.json              # Main Tauri config
├── tauri.dev.conf.json          # Dev overrides
├── Info.plist                   # macOS metadata
├── entitlements.plist           # Sandbox permissions
├── capabilities/default.json    # Tauri v2 permissions
├── icons/                       # App icons (icns, png)
├── resources/web-dist/          # Bundled UI (from Vite build)
└── sidecars/                    # Compiled OpenCode binary (Bun compile)

scripts/
├── build-sidecar.mjs            # Compile OpenCode server to binary
├── dev-web-server.mjs           # Dev server (port 3901)
└── desktop-dev.mjs              # Orchestrate dev environment

public/                          # Static fonts (IBM Plex Mono)
noop-dist/                       # Placeholder for builds
```

**IPC Commands (Tauri 2 invoke):**
- **Window:** new_window, new_window_at_url, set_window_theme, restart
- **Files:** open_path, open_in_app, read_file
- **Apps:** get_installed_apps, filter_installed_apps, fetch_app_icons
- **Hosts:** hosts_get, hosts_set, host_probe
- **Notifications:** notify
- **Updates:** check_for_updates, download_and_install_update
- **SSH:** ssh_instances_get, ssh_instances_set, ssh_connect, ssh_disconnect, ssh_status, ssh_logs

**Key Dependencies:**
- tauri 2.9.4 (core framework)
- window-vibrancy 0.7.1 (macOS effects)
- tokio 1.38 (async runtime)
- reqwest 0.12.4 (HTTP client)
- serde/serde_json 1.0.x (serialization)

**Sidecar Build:**
```
build-sidecar.mjs:
  bun build --compile packages/web/server/index.js
    → Targets: aarch64-apple-darwin, x86_64-apple-darwin
    → Output: src-tauri/sidecars/openchamber-server-{target}
    → Chmod 755

App Bundle:
  externalBin: ["sidecars/openchamber-server"]
  resources: ["resources/web-dist/**/*"]
    → Compiled + bundled into .app
```

**Runtime Flow:**
```
App Launch
  → Show splash window
  → Spawn sidecar (or connect to dev server)
  → Health check loop (reqwest, 5s intervals)
  → Probe remote hosts (soft 5s, hard 30s)
  → Navigate webview to URL
  → Load UI + inject init script
```

---

### 4. packages/vscode (VS Code Extension)
**Purpose:** Editor-native extension providing chat sidebar, agent manager, session editor, file integration.

**Key Stats:**
- **22 TypeScript modules** (~900 lines bridge.ts alone)
- **647-line extension.ts** entry point
- **Bridge Protocol:** RPC messages between extension & webview
- **VS Code 1.85+** minimum

**Directory Structure:**
```
src/
├── extension.ts          # VS Code extension host, command registration
├── opencode.ts           # OpenCode client, session management
├── bridge.ts             # RPC bridge protocol (1000+ lines)
├── webview/              # Webview API (React + @openchamber/ui)
│   └── (Vite-built React SPA)
├── modules/              # Extension modules (22 total)
│   ├── terminal.ts
│   ├── git.ts
│   ├── files.ts
│   ├── editor-operations.ts
│   ├── context.ts
│   └── ...
└── types/                # TypeScript definitions

media/                    # Icons, extension.jpg
```

**Bridge Protocol:**
- `api:proxy` — Forward HTTP requests
- `api:session:message` — Send message to OpenCode
- `api:sse:*` — SSE event streams
- `api:config/*` — Config operations
- `api:vscode/*` — VS Code-specific actions
- `command` — Execute VS Code commands
- `themeChange` — Theme synchronization

**Key Features:**
- Editor-native: open files from tool output, integrated editor panel
- Agent Manager: parallel multi-model runs
- Right-click actions: add context, explain, improve code
- Theme sync: maps VS Code theme to OpenChamber theme
- Session persistence: remember open sessions across restarts

---

## Root Configuration

```
.
├── package.json                # Bun workspaces config
├── tsconfig.json               # TypeScript project references
├── vite.config.ts              # Root Vite config (React, theme plugin)
├── eslint.config.js            # ESLint 9 + typescript-eslint
├── Dockerfile                  # Multi-stage Docker build
├── docker-compose.yml          # Docker Compose setup
├── .github/workflows/           # CI/CD
│   ├── release.yml             # macOS DMG release
│   ├── vscode-extension.yml    # VS Code publishing
│   └── oc-integration.yml      # OpenCode integration test
└── scripts/
    ├── install.sh              # Installation script
    ├── release/                # Release automation
    └── version/                # Version management
```

**Key Configs:**
- **Vite:** React plugin, theme storage plugin, manual chunk splitting (vendor-react, vendor-zustand, etc.)
- **TypeScript:** Strict mode, bundler module resolution, path aliases
- **ESLint:** 9 with typescript-eslint + react-hooks + react-refresh
- **PostCSS:** TailwindCSS v4 with design tokens

**Scripts:**
- `dev:web:full` — Full web dev (server + frontend)
- `dev:web:hmr` — Frontend HMR only
- `desktop:dev` — Desktop app dev
- `vscode:dev` — VS Code extension dev
- `release:prepare` — Prepare release artifacts
- `version:bump` — Update version

---

## Monorepo Dependency Graph

```
packages/ui (core)
  ├── React 19, TypeScript, Tailwind, Zustand
  └── No dependencies on other packages

packages/web (depends on ui)
  ├── @openchamber/ui (src alias)
  ├── Express, ws, simple-git, @octokit/rest
  └── Frontend: ui + Vite build

packages/desktop (depends on web)
  ├── Tauri 2 + Rust
  ├── Sidecar: compiled packages/web/server/index.js
  └── Frontend: bundled packages/ui via resources/web-dist/

packages/vscode (depends on ui)
  ├── VS Code API
  ├── @openchamber/ui (Vite-built webview)
  └── Bridge protocol to spawn local web server
```

---

## Build Artifacts

### Development
- **Vite HMR:** http://127.0.0.1:5173 (hot reload)
- **Backend:** http://127.0.0.1:3001 (Express server)
- **Desktop:** Tauri window (http://127.0.0.1:3901 sidecar or HMR fallback)
- **VS Code:** Webview (Vite-built bundle)

### Production
- **Web:** npm package (@openchamber/web), installable as CLI
- **Desktop:** .app bundle + DMG installer (macOS)
- **VS Code:** .vsix extension package
- **Docker:** Multi-stage image with OpenCode + CloudFLared sidecar

---

## Key Entry Points

| Package | Entry | Language |
|---------|-------|----------|
| ui | src/main.tsx | TypeScript/React |
| web (frontend) | src/main.tsx | TypeScript/React |
| web (server) | server/index.js | JavaScript (Node/Bun) |
| web (CLI) | bin/cli.js | JavaScript |
| desktop | src-tauri/src/main.rs | Rust |
| desktop (web) | src-tauri/resources/web-dist/index.html | HTML |
| vscode | src/extension.ts | TypeScript |
| vscode (webview) | src/webview/index.tsx | TypeScript/React |

---

## File Size Management

**Target:** Keep code files under 200 lines for optimal context.

**Current Large Files:**
- server/index.js: 14,413 lines (monolithic; refactoring planned)
- main.rs: 3,400 lines (platform code; acceptable)
- ChatInput.tsx: 28K tokens (refactoring candidate)
- bridge.ts: 32K tokens (protocol; acceptable)

**Splitting Strategy:** Future refactors will break server into lib/ modules (opencode, tunnels, git, github, terminal, etc.).

---

## Technology Stack Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| **Language** | TypeScript | 5.8.3 |
| **UI Framework** | React | 19.1.1 |
| **State** | Zustand | 5.0.8 |
| **Styling** | Tailwind CSS | 4.0 |
| **Build (web)** | Vite | 7.1.2 |
| **Server** | Express | 5.1.0 |
| **Desktop** | Tauri | 2.9.4 |
| **Desktop (Rust)** | Edition 2021 | Latest |
| **Package Manager** | Bun | 1.3.5+ |
| **Linter** | ESLint 9 | + typescript-eslint |
| **Editor** | CodeMirror 6 | Latest |
| **Terminal** | Ghostty Web | 0.3.0 |
| **Diff Viewer** | Pierre | 1.1.0-beta.13 |

---

## Cross-Package Communication

1. **UI → Web API** — HTTP + WebSocket (fetch, ws)
2. **Web Frontend → Server** — REST API (100+ endpoints)
3. **Desktop → Sidecar** — HTTP + WebSocket (local only)
4. **Desktop → Native** — Tauri IPC (invoke)
5. **VS Code → Webview** — postMessage bridge
6. **Web → OpenCode** — HTTP proxy + SSE

---

## Security Boundaries

- **UI Password:** Optional; defaults disabled
- **Tunnel Tokens:** One-time use per session
- **OAuth:** Device flow only; tokens in ~/.opencode/auth.json (mode 600)
- **Local Binding:** 127.0.0.1 by default; explicit --host for LAN
- **SSH:** User-controlled; no auto-discovery
- **Path Validation:** Prevent directory traversal, symlink detection
- **CORS:** Limited to local origins in development

---

## Deployment Targets

1. **npm global:** `npm install -g @openchamber/web` → openchamber CLI
2. **Docker:** Multi-stage build with OpenCode + CloudFlared
3. **macOS App:** .app bundle + DMG (auto-updater via GitHub releases)
4. **VS Code Marketplace:** Published .vsix extension
5. **Bun compile:** Standalone binary for serverless/embedded

---

**Last Updated:** 2026-03-24 | **Maintainer:** OpenChamber Team | **License:** MIT
