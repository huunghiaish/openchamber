# OpenCode CLI Integration Scout Report

## Executive Summary

OpenChamber integrates OpenCode via:
1. **HTTP proxy layer** - Web server acts as reverse proxy to local OpenCode instance
2. **SSE (Server-Sent Events)** for real-time message/event streaming
3. **SDK client** (`@opencode-ai/sdk/v2`) for type-safe API access
4. **Managed process spawning** on startup or external server URL mode

## 1. Process Management

### OpenCode Binary Spawning

**File:** `/Users/nghia/Projects/openchamber/packages/web/server/index.js`

#### Spawn Logic (line 5934-6050)
- **Function:** `createManagedOpenCodeServerProcess({ hostname, port, timeout, cwd, env })`
- **Binary resolution:** Reads `OPENCODE_BINARY` env var or uses `'opencode'` CLI
- **Command:** `opencode serve --hostname <hostname> --port <port>`
- **Platform support:**
  - **Windows WSL mode:** Uses WSL wrapper to run OpenCode in WSL distro
  - **Windows native:** Detects shim interpreters (Node/Bun shebangs) and spawns via correct interpreter
  - **Unix:** Direct spawn via Node's `child_process.spawn()`
- **Stdio handling:** `['ignore', 'pipe', 'pipe']` - captures stdout/stderr for URL detection
- **Output parsing:** Extracts URL from process stdout to detect when server is ready

#### Startup Flow (line 6101-6161)
- **Function:** `startOpenCode()`
- **Steps:**
  1. Resolve port (env-configured or OS-allocated)
  2. Apply binary from settings
  3. Set up OpenCode CLI environment
  4. Generate/rotate `OPENCODE_SERVER_PASSWORD` for local auth
  5. Spawn process via `createManagedOpenCodeServerProcess()`
  6. Wait for health check (10s timeout)
  7. Extract API prefix from startup URL
  8. Mark as ready

#### Configuration (lines 3745-3815)
- **Port:** `OPENCODE_PORT` or `OPENCHAMBER_OPENCODE_PORT` env vars
- **Host:** `OPENCODE_HOST` (full base URL, e.g., `http://hostname:4096`)
- **Skip startup:** `OPENCODE_SKIP_START=true` or `OPENCHAMBER_SKIP_OPENCODE_START=true`
- **WSL distro:** `OPENCODE_WSL_DISTRO` or `OPENCHAMBER_OPENCODE_WSL_DISTRO`
- **Password:** `OPENCODE_SERVER_PASSWORD` (auto-generated if local)

## 2. HTTP Proxy Layer (OpenCode API Forwarding)

### Architecture: Reverse Proxy Pattern

**File:** `/Users/nghia/Projects/openchamber/packages/web/server/index.js` (lines 6783-7009)

#### Request Forwarding

- **Generic handler:** `forwardGenericApiRequest(req, res)` (line 6852)
  - Translates UI `/api/*` → OpenCode server (`http://localhost:<port>/...`)
  - Collects request headers, filters hop-by-hop headers
  - Adds auth headers (`OPENCODE_SERVER_PASSWORD` → `Authorization` header) if needed
  - Forwards request body (up to 50MB)
  - Streams response back with proper status codes

- **Dedicated large message handler:** `/api/session/:sessionId/message` POST (line 6890)
  - Handles multi-file attachments up to 50MB
  - Separate logic to avoid streaming edge cases with large payloads

- **SSE forwarding:** `forwardSseRequest()` (line 6761)
  - Routes `/api/event` and `/api/global/event` → OpenCode SSE endpoints
  - Maintains event stream for real-time updates

#### Exception Routes (Not Forwarded)
```
/api/themes/custom
/api/config/agents
/api/config/opencode-resolution
/api/config/settings
/api/config/skills
/api/health
/api/session/message (handled separately)
```

#### URL Construction (line 5061-5070)
```javascript
function buildOpenCodeUrl(path, prefixOverride) {
  // Base: openCodeBaseUrl ?? `http://localhost:${openCodePort}`
  // Prefix: API prefix (e.g., "/sdk" if OpenCode exposes at /sdk/session)
  // Full: `${base}${prefix}${path}`
}
```

## 3. Client-Side SDK Integration

### SDK Client Wrapper

**File:** `/Users/nghia/Projects/openchamber/packages/ui/src/lib/opencode/client.ts`

- **Import:** `createOpencodeClient` from `@opencode-ai/sdk/v2`
- **Base URL:** Default `/api` (relative to web server, proxied to OpenCode)
- **Override:** `VITE_OPENCODE_URL` env var for absolute URLs
- **Singleton:** `opencodeClient` instance exported and used by stores

### Type Imports from SDK
```typescript
Session, Message, Part, Provider, Config, Model, Agent, 
TextPartInput, FilePartInput, Event
```

## 4. Event Streaming (SSE)

### Protocol

**File:** `/Users/nghia/Projects/openchamber/packages/ui/src/hooks/useEventStream.ts`

- **Transport:** Server-Sent Events (SSE) over HTTP GET
- **Endpoints:** 
  - `/api/event` → `<opencode>/event` (session-scoped)
  - `/api/global/event` → `<opencode>/global/event` (global events)
- **Event format:** Server sends SSE `data:` lines with JSON payloads
- **Hook:** `useEventStream()` manages subscription lifecycle

### Message & Tool Flow

**Files:**
- `useSessionStore.ts` - State management (Zustand)
- `useEventStream.ts` - SSE subscription handler
- `permissionStore.ts` - Permission request handling

**Event types observed:**
- `session.created`, `session.updated`, `session.status`
- `message.created`, `message.updated`, `message.part.updated`
- `tool.created`, `tool.updated`
- `permission.required`
- `todo.updated`

## 5. Abstraction Layers & Provider Patterns

### OpenCode Module (Config/Auth/Skills Management)

**Directory:** `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/`

**Submodules:**
- **`auth.js`** - Provider auth file ops (`~/.local/share/opencode/auth.json`)
- **`shared.js`** - Config layers (user/project/custom), markdown file ops, skill discovery
- **`ui-auth.js`** - UI session auth with rate limiting (separate from OpenCode server auth)
- **`commands.js`** - OpenCode commands (custom CLI extensions) CRUD
- **`agents.js`** - OpenCode agents CRUD
- **`skills.js`** - OpenCode skills discovery & management
- **`providers.js`** - Provider config (model/API key management)
- **`mcp.js`** - MCP (Model Context Protocol) server config

**No abstraction for swapping backends:**
- OpenCode is tightly coupled (HTTP proxy, SSE, SDK types)
- No provider interface for alternative AI backends
- Provider management is OpenCode config, not backend abstraction

### Skills-Catalog Module (Parallel System)

**Directory:** `/Users/nghia/Projects/openchamber/packages/web/server/lib/skills-catalog/`

- Manages custom skills (separate from OpenCode built-in agents)
- Discovers & installs from git, local FS, ClawdHub registry
- Integrates with UI settings (`/components/sections/skills/`)

## 6. Key Files Summary

### Server-Side

| Path | Purpose |
|------|---------|
| `packages/web/server/index.js` | Express server, OpenCode spawn, HTTP proxy, SSE relay |
| `packages/web/server/lib/opencode/` | Config/auth/skills management, UI auth |
| `packages/web/server/lib/package-manager.js` | OpenCode binary installer/resolver |
| `packages/web/server/lib/cloudflare-tunnel.js` | Tunnel support for remote access |
| `packages/web/bin/cli.js` | CLI entry point |

### Client-Side

| Path | Purpose |
|------|---------|
| `packages/ui/src/lib/opencode/client.ts` | SDK client singleton (HTTP base URL resolution) |
| `packages/ui/src/hooks/useEventStream.ts` | SSE subscription handler |
| `packages/ui/src/stores/useSessionStore.ts` | Session/message state (Zustand) |
| `packages/ui/src/stores/permissionStore.ts` | Permission request handling |
| `packages/ui/src/stores/sessionStore.ts` | Legacy session state |
| `packages/ui/src/stores/messageStore.ts` | Message state |

### Architecture Reference

| Path | Purpose |
|------|---------|
| `AGENTS.md` | Project architecture, tech stack, constraints (MANDATORY read) |
| `packages/web/server/lib/opencode/DOCUMENTATION.md` | OpenCode module API docs |

## 7. Communication Flow

```
UI (React/Zustand)
    ↓ HTTP (fetch/axios)
Web Server Express `/api/*` routes (proxy)
    ↓ HTTP + Basic Auth (OPENCODE_SERVER_PASSWORD)
OpenCode Server (spawned or external)
    ├─ POST /session/:id/message → Tool execution, code gen
    ├─ GET /event (SSE) → Real-time message updates
    ├─ GET /session → List sessions
    └─ GET /config, /agent, /model, /provider → Metadata
```

## 8. Environment Configuration

**For external OpenCode server:**
```bash
OPENCODE_HOST=http://hostname:4096          # Full URL
OPENCODE_SKIP_START=true                    # Don't spawn
OPENCODE_SERVER_PASSWORD=secret             # Auth password
```

**For managed spawn:**
```bash
OPENCODE_PORT=4096                          # Preferred port (or OS-allocated)
OPENCHAMBER_OPENCODE_HOSTNAME=127.0.0.1     # Bind address
OPENCODE_BINARY=/path/to/opencode           # Binary override
OPENCODE_WSL_DISTRO=Ubuntu                  # WSL distro on Windows
```

## Key Insights

1. **No pluggable backend** - OpenCode is assumed. HTTP proxy layer is for UI ↔ OpenCode routing, not for swapping backends.
2. **Tight SDK coupling** - Types imported directly from `@opencode-ai/sdk/v2`; no abstraction layer.
3. **Password-based auth** - `OPENCODE_SERVER_PASSWORD` environment variable for local server security.
4. **SSE for streaming** - Real-time events (messages, permissions, tool output) via `/event` endpoints.
5. **Dual auth** - OpenCode server auth (password) separate from UI session auth (cookies/tokens).
6. **Process management** - Cross-platform spawn (Windows WSL shim detection, Unix direct spawn).
7. **Skills as parallel system** - Custom agent skills managed separately from OpenCode config.

## Unresolved Questions

None - architecture is documented and consistent.

---

## Appendix: Detailed File Listing

### Server OpenCode Integration Files

**Core Server:**
- `/Users/nghia/Projects/openchamber/packages/web/server/index.js` (14,413 lines)
  - OpenCode process spawn: lines 5934-6161 (`createManagedOpenCodeServerProcess`, `startOpenCode`)
  - HTTP proxy setup: lines 6783-7009 (`forwardGenericApiRequest`, `forwardSseRequest`)
  - Config/auth: lines 3745-3865 (environment vars, password management)
  - Health checks: lines 3696-3719, 6264-6437

**OpenCode Module:**
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/index.js` - Public entrypoint (exports from submodules)
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/auth.js` - Provider auth file ops
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/shared.js` - Config layers, markdown parsing, skill discovery
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/ui-auth.js` - UI session auth with rate limiting
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/commands.js` - OpenCode commands CRUD
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/agents.js` - OpenCode agents CRUD
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/skills.js` - Skills discovery & management
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/providers.js` - Provider config management
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/mcp.js` - MCP server config
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/DOCUMENTATION.md` - Module API reference

**Supporting Modules:**
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/package-manager.js` - Binary installer/resolver
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/cloudflare-tunnel.js` - Tunnel support
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/terminal/` - WebSocket terminal I/O
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/notifications/` - Desktop notifications
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/skills-catalog/` - Custom skills discovery

**CLI Entry:**
- `/Users/nghia/Projects/openchamber/packages/web/bin/cli.js` - CLI invocation point

### Client OpenCode Integration Files

**SDK Client & Hooks:**
- `/Users/nghia/Projects/openchamber/packages/ui/src/lib/opencode/client.ts` (lines 1-80+)
  - SDK client factory with base URL resolution
  - Type imports from `@opencode-ai/sdk/v2`
  - Export: `opencodeClient` singleton

- `/Users/nghia/Projects/openchamber/packages/ui/src/hooks/useEventStream.ts`
  - SSE subscription handler
  - Event parsing (message, tool, permission events)
  - Integration with Zustand stores

**State Management (Zustand):**
- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/useSessionStore.ts` (lines 1-80+)
  - Session/message/context state
  - Integration with `opencodeClient`
  - Uses `useEventStream()` hook

- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/sessionStore.ts` - Legacy session state
- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/messageStore.ts` - Message state
- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/permissionStore.ts` - Permission request handling
- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/useConfigStore.ts` - Config state
- `/Users/nghia/Projects/openchamber/packages/ui/src/stores/useUIStore.ts` - UI state (event stream status)

**Chat UI Components:**
- `/Users/nghia/Projects/openchamber/packages/ui/src/components/chat/` - Chat interface components
  - `TurnItem.tsx`, `TurnList.tsx` - Message rendering
  - `MobileAgentButton.tsx`, `MobileModelButton.tsx` - Agent/model selectors
  - `PermissionRequest.tsx` - Permission flow UI

**Settings & Config UI:**
- `/Users/nghia/Projects/openchamber/packages/ui/src/components/sections/openchamber/` - OpenCode config UI
- `/Users/nghia/Projects/openchamber/packages/ui/src/components/sections/skills/` - Skills management UI

### Documentation & Architecture

- `/Users/nghia/Projects/openchamber/AGENTS.md` - **MANDATORY** read for architecture, tech stack, constraints
- `/Users/nghia/Projects/openchamber/packages/web/server/lib/opencode/DOCUMENTATION.md` - OpenCode module API
- `/Users/nghia/Projects/openchamber/packages/web/server/TERMINAL_INPUT_WS_PROTOCOL.md` - WebSocket protocol for terminal

---

## Architecture Diagram (ASCII)

```
┌─────────────────────────────────────────────────────────────┐
│ Browser (React UI)                                          │
│                                                              │
│  useSessionStore() ──┐                                       │
│  useEventStream() ───┤  useOpencodeClient()                 │
│  permissionStore ────┘      (SDK client)                    │
└────────────┬─────────────────────┬───────────────────────────┘
             │ HTTP (fetch/axios)  │ SSE
             │                     │
    ┌────────▼─────────────────────▼──────────────┐
    │ OpenChamber Web Server (Express)            │
    │ packages/web/server/index.js                │
    │                                             │
    │  /api/* routes (reverse proxy)             │
    │  └─ forwardGenericApiRequest()             │
    │  /api/event (SSE streaming)                │
    │  └─ forwardSseRequest()                    │
    │  /api/session/:id/message (large)          │
    │  └─ dedicated handler                      │
    │                                             │
    │  Exception routes:                          │
    │  /api/config/*  (local handlers)           │
    │  /api/skills/*  (custom skills)            │
    │  /auth/*        (UI session auth)          │
    └─┬──────────────────────────────────────────┘
      │ HTTP + Authorization: Bearer <password>
      │ (buildOpenCodeUrl → http://localhost:PORT/...)
      │
    ┌─▼────────────────────────────────────┐
    │ OpenCode Server (spawned or external) │
    │ Command: opencode serve --port <PORT>│
    │                                       │
    │ HTTP Endpoints:                       │
    │  POST /session/:id/message           │
    │  GET  /session (list)                │
    │  GET  /event (SSE stream)            │
    │  GET  /global/event (SSE stream)     │
    │  GET  /config                        │
    │  GET  /agent, /model, /provider      │
    └───────────────────────────────────────┘
```

**Process Management:**

```
OpenChamber Startup
    ↓
startOpenCode()
    ├─ resolve port (env or OS-allocated)
    ├─ loadBinaryPath() → 'opencode' or OPENCODE_BINARY
    ├─ generatePassword() → OPENCODE_SERVER_PASSWORD
    ├─ spawn(binary, ['serve', '--hostname', hostname, '--port', port])
    │  └─ (Windows WSL mode: wrap in wsl.exe call)
    │  └─ (Windows native: detect shim, use correct interpreter)
    ├─ parseStdout() → extract URL & port
    ├─ waitForReady() → health check (/health endpoint)
    └─ setOpenCodePort() → make available to proxy
```

**Environment Configuration Priority:**

```
External Server (skip spawn):
  1. OPENCODE_HOST=<full URL> (takes precedence)
  2. OPENCODE_PORT=<port> + localhost resolve
  3. OPENCODE_SKIP_START=true

Managed Spawn:
  1. OPENCODE_PORT (preferred port, 0 = OS-allocate)
  2. OPENCHAMBER_OPENCODE_HOSTNAME (bind address)
  3. OPENCODE_BINARY (path to opencode CLI)
  4. OPENCODE_WSL_DISTRO (Windows: WSL distro name)

Auth:
  OPENCODE_SERVER_PASSWORD (auto-generated if local, can be overridden)
```

