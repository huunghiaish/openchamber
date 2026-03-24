# Scout Report: OpenChamber Web Package

**Date:** 2026-03-24
**Scope:** packages/web/ (Web server & PWA entry point)
**Package Name:** @openchamber/web v1.9.1

---

## 1. Tech Stack & Dependencies

### Frontend (Client-side)
- **React 19.1.1** - UI framework
- **Vite 7.1.2** - Build tool and dev server
- **TypeScript 5.8.3** - Type checking
- **TailwindCSS 4.0.0** - Styling (@tailwindcss/postcss)
- **Radix UI** - Accessible component primitives (@radix-ui/*)
- **CodeMirror 6** - Code editor (lang-cpp, lang-go)
- **React Markdown 10.1.0** + remark-gfm - Markdown rendering
- **Zustand 5.0.8** - State management
- **next-themes 0.4.6** - Theme management
- **Sonner 2.0.7** - Toast notifications
- **React Syntax Highlighter 15.6.6** - Code syntax highlighting
- **Ghostty Web 0.3.0** - Terminal emulation UI

### Backend (Server-side)
- **Express 5.1.0** - HTTP server
- **Node.js 20+** or Bun - Runtime
- **ws 8.18.3** - WebSocket support
- **http-proxy-middleware 3.0.5** - HTTP proxy for OpenCode
- **web-push 3.6.7** - Push notification sending
- **node-pty 1.2.0-beta.12** - PTY support for terminal
- **bun-pty 0.4.5** - Bun PTY alternative
- **Jose 6.1.3** - JWT/JWE handling

### Build & Dev Tools
- **Vite PWA Plugin 1.0.3** - PWA & Service Worker generation
- **workbox-window 7.4.0** - Service worker client
- **Autoprefixer 10.4.21** - CSS vendor prefixing
- **ESLint 9.33.0** - Code linting
- **Nodemon 3.1.7** - Dev server auto-reload

### Additional Dependencies
- **@opencode-ai/sdk v1.3.0** - OpenCode SDK client
- **@octokit/rest 22.0.1** - GitHub API client
- **simple-git 3.28.0** - Git operations
- **adm-zip 0.5.16** - ZIP file handling
- **qrcode-terminal 0.12.0** - QR code generation
- **yaml 2.8.1** - YAML parsing
- **@clack/prompts 1.1.0** - CLI prompts
## 2. Directory Structure

```
packages/web/
├── bin/                      # CLI entry point
│   ├── cli.js               # openchamber command
│   └── cli-*.js             # CLI support modules
│
├── src/                     # Frontend (React/TypeScript)
│   ├── main.tsx             # App entry, PWA registration
│   ├── sw.ts                # Service Worker
│   ├── pwa.d.ts             # PWA type definitions
│   └── api/                 # Runtime API implementations
│       ├── index.ts         # API factory creating RuntimeAPIs
│       ├── terminal.ts      # Terminal API wrapper
│       ├── git.ts           # Git API (HTTP proxies)
│       ├── files.ts         # File system API
│       ├── settings.ts      # Settings/config API
│       ├── permissions.ts   # Permission checks
│       ├── notifications.ts # Notification API
│       ├── github.ts        # GitHub API client
│       ├── push.ts          # Web Push API
│       └── tools.ts         # Tools API
│
├── server/                  # Backend (Express)
│   ├── index.js             # Main server (14,413 lines)
│   ├── index.d.ts           # TypeScript definitions
│   └── lib/
│       ├── opencode/        # OpenCode integration
│       ├── tunnels/         # Cloudflare tunnel management
│       ├── terminal/        # Terminal PTY handling
│       ├── git/             # Git utilities & service
│       ├── github/          # GitHub integration
│       ├── quota/           # Model quota tracking (20+ providers)
│       ├── skills-catalog/  # Skills management & ClawdHub
│       ├── tts/             # Text-to-speech service
│       ├── notifications/   # Notification management
│       └── ...
│
├── public/                  # Static assets (icons, manifest)
├── vite.config.ts           # Vite configuration
├── tsconfig.json            # TypeScript config
├── index.html               # HTML entry point
├── package.json             # Dependencies & scripts
└── README.md
```

---

## 3. Server Architecture (server/index.js - 14,413 lines)

### Startup Flow
```
bin/cli.js serve
  → Parse args (--port, --host, --ui-password)
  → Invoke main() from server/index.js
    → Create Express app
    → Setup middleware (JSON parsing, logging, auth)
    → Register 100+ API routes
    → Create HTTP server
    → Listen on port (default 3000, binds to 127.0.0.1)
    → Start/connect to OpenCode subprocess
    → Setup WebSocket servers (terminal, notifications)
    → Graceful shutdown handlers
```

### Key Components

#### 1. OpenCode Integration
- Auto-spawns OpenCode subprocess (configurable via OPENCODE_PORT/OPENCODE_HOST)
- Proxies /api/opencode/* requests to OpenCode server
- Handles auth between web server and OpenCode
- Detects OpenCode binary (platform-specific)
- Settings/credentials management

#### 2. Tunnel Management (Cloudflare)
- Three modes: quick (temporary), managed-local, managed-remote
- Cloudflare tunnel provider
- Stores config in ~/.config/openchamber/tunnel-cli-state.json
- One active tunnel per OpenChamber instance
- Handles tunnel auth & connection tokens

#### 3. Terminal Support
- PTY-based terminal sessions via WebSocket
- Input path: /api/terminal/input
- Terminal session creation, resize, input, output streaming
- Uses node-pty or bun-pty for cross-platform support
- Ghostty Web terminal UI frontend

#### 4. Authentication
- UI Auth: Optional password protection (--ui-password flag)
- Tunnel Auth: Per-session tokens for tunnel connections
- Provider Auth: OAuth/API key storage in ~/.opencode/auth.json
- Request scope classification: local, tunnel, unknown-public

#### 5. Push Notifications
- Web Push API integration (VAPID keys)
- Service Worker handles push events
- Endpoints: subscribe, unsubscribe, get VAPID public key

#### 6. Settings & Config
- User settings in ~/.config/openchamber/settings.json
- Persistent: workspace paths, projects, themes
- Settings validation & normalization
- Custom theme support in ~/.config/openchamber/themes/

#### 7. Skills & Agents
- Agents, commands, skills config management
- Skills discovery from filesystem and ClawdHub catalog
- Skills installation/removal
- MCP server configuration

---

## 4. API Endpoints (100+)

**System:**
- GET /health - Server health check
- POST /api/system/shutdown - Graceful shutdown
- GET /api/system/info - Server info (version, PID, uptime)
- GET /connect - Token-based connect link

**Sessions & Activity:**
- GET /api/sessions/snapshot - Active sessions
- GET /api/sessions/:id/status - Session status
- POST /api/sessions/:id/message - Forward to OpenCode
- GET /api/session-activity - Activity tracking

**Config:**
- GET /api/config/settings - Get settings
- PUT /api/config/settings - Update settings
- GET /api/config/themes - Get themes
- GET /api/config/opencode-resolution - OpenCode config

**File System:**
- GET /api/fs/list - List directory
- POST /api/fs/search - Search files

**Git:**
- GET /api/git/status - Git status
- POST /api/git/commit - Create commit
- POST /api/git/push - Push changes

**GitHub:**
- GET /api/github/auth/status - Auth status
- POST /api/github/auth/start - Device flow start
- POST /api/github/auth/complete - Device flow complete
- GET /api/github/repos - List repos
- POST /api/github/pr - Create PR

**Terminal:**
- POST /api/terminal/create - Create PTY session
- WS /ws/terminal/:sessionId - Terminal stream
- POST /api/terminal/:sessionId/input - Send input

**OpenCode:**
- GET /api/event - SSE event stream
- POST /api/session/:sessionId/message - Message to OpenCode
- GET /api/openchamber/models-metadata - Model metadata

**Tunnels:**
- GET /api/openchamber/tunnel/status - Tunnel status
- POST /api/openchamber/tunnel/start - Start tunnel
- POST /api/openchamber/tunnel/stop - Stop tunnel
- GET /api/openchamber/tunnel/check - Check tunnel availability

**TTS & Voice:**
- POST /api/tts/speak - Text-to-speech
- POST /api/voice/token - Voice token

**Push Notifications:**
- GET /api/push/vapid-public-key
- POST /api/push/subscribe
- DELETE /api/push/subscribe
- GET /api/push/visibility

**Updates:**
- GET /api/openchamber/update-check - Check for updates
- POST /api/openchamber/update-install - Install update

---

## 5. Frontend Architecture

### Entry Point: src/main.tsx
```typescript
// 1. Create web APIs (terminal, git, files, settings, etc.)
window.__OPENCHAMBER_RUNTIME_APIS__ = createWebAPIs();

// 2. Register Service Worker (PWA in production)
if (import.meta.env.PROD) {
  registerSW({ onRegisterError });
} else {
  // Dev: unregister old SWs
}

// 3. Import UI shell from @openchamber/ui
import(@''openchamber/ui/main\);
```

### Web API Layer (src/api/)
Web-specific implementations of RuntimeAPIs (contract with @openchamber/ui):

1. **Terminal API** - Wraps HTTP calls to /api/terminal/* endpoints
2. **Git API** - HTTP proxies to server Git endpoints (from gitApiHttp)
3. **Files API** - HTTP calls to /api/fs/list, /api/fs/search
4. **Settings API** - HTTP calls to /api/config/settings
5. **Permissions API** - Check permissions via /api/permissions/*
6. **Notifications API** - Web Notification API + Tauri fallback
7. **GitHub API** - HTTP calls to /api/github/*
8. **Push API** - Web Push API integration
9. **Tools API** - Tool execution endpoints

### Service Worker (src/sw.ts)

**Purpose:** Minimal PWA support for push notifications
- Handles push events (shows notifications)
- Handles notificationclick events (opens URL)
- Skips Workbox runtime helpers for iOS reliability
- Uses IIFE bundle format for iOS Safari compatibility

**PWA Configuration (vite.config.ts):**
- Strategy: injectManifest (custom SW)
- Precache patterns: **/*.{js,css,html,ico,png,svg,woff,woff2,ttf,otf,eot}
- Auto-update registration
- iOS-friendly manifest

---

## 6. Build & Development

### Build Configuration (vite.config.ts)

**Key Settings:**
- Entry: src/ (React app)
- Output: dist/ (production)
- Plugins:
  - React 19 with Babel compiler plugin
  - Vite PWA plugin (custom SW + precache)
  - Theme storage plugin (persist theme preference)

**Dev Server:**
- Port: 5173 (default Vite)
- Proxies to backend:
  - /auth → http://127.0.0.1:3001
  - /api → http://127.0.0.1:3001
  - /health → http://127.0.0.1:3001

**Code Splitting:**
- Vendor chunks: React, Zustand, OpenCode SDK, Radix UI, Markdown, Syntax highlighter
- Chunk size warning limit: 1200 KB

**Aliases:**
- @openchamber/ui → ../ui/src
- @web → ./src
- @opencode-ai/sdk/v2 → custom dist path

### Scripts

```bash
npm run dev                   # Watch & rebuild (Vite)
npm run dev:server          # Run server only
npm run dev:server:watch    # Server with auto-reload
npm run build               # Vite production build
npm run build:watch         # Watch mode for CI/builds
npm run type-check          # TypeScript check
npm run lint                # ESLint check
npm run start               # Run CLI (alias for bin/cli.js serve)
```

---

## 7. OpenCode CLI Integration

### CLI Entry Point: bin/cli.js

**Commands:**

1. **serve** (default)
   ```bash
   openchamber [--port 3000] [--host 127.0.0.1] [--ui-password secret] [--foreground]
   ```

2. **tunnel** - Cloudflare tunnel management
   ```bash
   openchamber tunnel start [--provider cloudflare] [--mode quick|managed-local|managed-remote]
   openchamber tunnel stop [--port 3000]
   openchamber tunnel status [--all]
   openchamber tunnel profile add --provider cloudflare --mode managed-remote ...
   openchamber tunnel providers
   openchamber tunnel help
   ```

3. **logs** - View server logs
   ```bash
   openchamber logs [--tail 200] [--follow]
   ```

4. **update** - Check/install updates
   ```bash
   openchamber update
   openchamber update check
   ```

5. **stop** - Graceful shutdown
   ```bash
   openchamber stop [--port 3000]
   ```

### Environment Variables

**OpenCode Control:**
```bash
OPENCODE_PORT=4096              # Connect to external OpenCode server
OPENCODE_HOST=https://myhost:4096  # Override OpenCode URL
OPENCODE_SKIP_START=true        # Dont start embedded OpenCode
OPENCHAMBER_OPENCODE_HOSTNAME=0.0.0.0  # Bind address for managed OpenCode
```

**Server Control:**
```bash
OPENCHAMBER_PORT=3001           # Server port (CLI --port overrides)
OPENCHAMBER_HOST=127.0.0.1      # Bind address (defaults to 127.0.0.1)
OPENCHAMBER_TUNNEL_MODE=quick   # Default tunnel mode
OPENCHAMBER_TUNNEL_PROVIDER=cloudflare
OPENCHAMBER_TUNNEL_HOSTNAME=app.example.com
OPENCHAMBER_TUNNEL_TOKEN=<token>
OPENCHAMBER_TUNNEL_CONFIG=/path/to/config.yml
OPENCHAMBER_DISABLE_PWA_DEV=1   # Disable PWA in dev
VITE_ENABLE_REACT_SCAN=1        # Enable React Scan profiling
```

**Auth:**
```bash
UI_PASSWORD=secret              # UI password protection
```

---

## 8. PWA & Offline Support

### Web App Manifest
- Dynamic manifest generation via /api/manifest (fallback in index.html)
- Custom PWA name support via query param ?pwa_name=...
- Recent sessions stored in localStorage

### Service Worker Features
- **Push notifications** - Handles push events, shows notifications
- **Notification clicks** - Routes to correct session on click
- **No offline support** - SW is minimal, focused on push only

### iOS Considerations
- IIFE bundle format (non-module) for compatibility
- Apple touch icons (180x180, 167x167, 152x152)
- Viewport settings prevent user zoom on iOS

---

## 9. Key Features

### Terminal Emulation
- **Frontend:** Ghostty Web terminal UI
- **Backend:** Node PTY with WebSocket stream
- **Cross-platform:** macOS, Linux, Windows (WSL support)
- **Responsive:** Terminal resize, input streaming

### Code Editor
- **CodeMirror 6** integration
- **Language support:** C++, Go (extensible)
- **Syntax highlighting** via react-syntax-highlighter

### Git Integration
- **Operations:** status, diff, commit, push, pull, fetch, branch management
- **Worktrees:** Create, delete, validate Git worktrees
- **Identities:** Store/manage multiple Git identities
- **Credentials:** Persistent credential storage

### GitHub Integration
- **Device Flow Auth** - OAuth without browser redirect
- **PR Management:** Create, update, merge PRs
- **Issue Tracking:** List, comment on issues
- **PR Status Tracking:** Detailed PR context

### Model Quota Tracking
- 20+ provider integrations (Claude, OpenAI, Google, Ollama Cloud, etc.)
- Real-time quota monitoring
- Provider-specific auth & API validation

### Notifications
- **Web Push API** - Background notifications
- **Web Notification API** - In-app notifications
- **Desktop notifications** (Tauri fallback)
- **Badge updates** - Unread message counts

---

## 10. Build & Deployment

### Production Build
Output structure:
- HTML (index.html)
- JS chunks (vendor-*.js, main-*.js)
- CSS (auto-imported in chunks)
- Service Worker (sw.js, compiled from sw.ts)
- Assets (icons, fonts)

### Serving
Development: npm run dev
Production: npm start (runs CLI serve)

### Deployment Approaches

1. Standalone Binary (recommended)
   - Install: bun add -g @openchamber/web
   - Run: openchamber command

2. Docker
   - Runs embedded OpenCode
   - Custom hostname via OPENCHAMBER_OPENCODE_HOSTNAME

3. External OpenCode Server
   - Set OPENCODE_PORT or OPENCODE_HOST
   - Set OPENCODE_SKIP_START=true

---

## 11. Performance & Optimization

Bundle Optimization:
- Manual chunk splits (React, Zustand, SDK, Radix, Markdown)
- Chunk size warnings at 1200 KB
- Worker format: ES modules

Dev Performance:
- React 19 Compiler plugin
- React Scan profiling (optional)
- Hot module reloading

Network:
- 50 MB upload/download limit
- 4-minute timeout for long requests
- SSE streams for real-time events

---

## 12. Project Integration

@openchamber/ui Dependency:
- Core UI components from ../ui/src
- Type contracts from @openchamber/ui/lib/api/types
- Entry point: @openchamber/ui/main

OpenCode SDK Integration:
- @opencode-ai/sdk/v2 client for inference
- Manifests as /api/session/:sessionId/message proxy
- Models metadata cached at startup

Monorepo Structure:
- packages/web/ (this package)
- packages/ui/ (shared UI)
- vite-theme-plugin (theme Vite plugin)
- Root eslint.config.js

---

## 13. Security Considerations

Authentication:
- Optional UI password (--ui-password)
- Tunnel session tokens (one-time use)
- Provider OAuth tokens in ~/.opencode/auth.json (mode 600)

Path Security:
- Workspace path validation (prevent directory traversal)
- Symlink detection & handling
- Per-project config isolation

Network:
- Binds to 127.0.0.1 by default (localhost only)
- Optional 0.0.0.0 binding for LAN (requires explicit config)
- Browser-unsafe port detection
- Cloudflare tunnel provides HTTPS/remote access

---

## Summary

**OpenChamber Web** is a full-stack web application that:

1. **Serves a React UI** (Vite build) for interactive coding
2. **Proxies to OpenCode** (spawned subprocess or external)
3. **Manages tunnels** (Cloudflare) for remote access
4. **Handles terminal I/O** via WebSocket PTY
5. **Integrates Git & GitHub** APIs
6. **Supports PWA** deployment (iOS, Android, desktop)
7. **Tracks model quotas** across 20+ providers
8. **Manages user config** (settings, themes, credentials)
9. **Provides CLI** for server control & tunnel management

**Key Tech:** Express + React + Vite + WebSocket + PTY + Cloudflare Tunnels

**Entry Points:**
- Browser: localhost:3000 (development: localhost:5173)
- CLI: openchamber command

**Output:** Production-ready PWA deployable on macOS, Linux, Windows, Docker, or serverless platforms.

---

## Key Files to Know

**Frontend Entry:**
- /Users/nghia/Projects/openchamber/packages/web/src/main.tsx - React entry
- /Users/nghia/Projects/openchamber/packages/web/src/api/index.ts - API factory

**Backend Entry:**
- /Users/nghia/Projects/openchamber/packages/web/server/index.js - Express server (14K lines)
- /Users/nghia/Projects/openchamber/packages/web/bin/cli.js - CLI handler

**Configuration:**
- /Users/nghia/Projects/openchamber/packages/web/vite.config.ts - Build config
- /Users/nghia/Projects/openchamber/packages/web/tsconfig.json - TypeScript config
- /Users/nghia/Projects/openchamber/packages/web/package.json - Dependencies

**OpenCode Integration:**
- /Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/ - OpenCode service
- /Users/nghia/Projects/openchamber/packages/web/server/lib/tunnels/ - Tunnel support
- /Users/nghia/Projects/openchamber/packages/web/server/lib/terminal/ - Terminal PTY

**API Implementations:**
- /Users/nghia/Projects/openchamber/packages/web/src/api/terminal.ts - Terminal wrapper
- /Users/nghia/Projects/openchamber/packages/web/src/api/git.ts - Git API
- /Users/nghia/Projects/openchamber/packages/web/src/api/github.ts - GitHub API
- /Users/nghia/Projects/openchamber/packages/web/src/api/notifications.ts - Notifications
- /Users/nghia/Projects/openchamber/packages/web/src/api/push.ts - Web Push

