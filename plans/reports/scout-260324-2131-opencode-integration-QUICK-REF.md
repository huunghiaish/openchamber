# OpenCode Integration - Quick Reference

## TL;DR

**OpenChamber = HTTP Proxy + SDK Client → OpenCode Server**

### Where OpenCode is Spawned
- **File:** `packages/web/server/index.js` line 5934-6161
- **Function:** `createManagedOpenCodeServerProcess()` + `startOpenCode()`
- **Command:** `opencode serve --hostname <hostname> --port <port>`
- **Cross-platform:** Windows WSL support, shim detection for Node/Bun wrappers

### How UI Talks to OpenCode
```
Browser
  ↓ HTTP fetch
Web Server (/api/* proxy)
  ↓ HTTP + password auth
OpenCode Server
```
- **Files:** `packages/web/server/index.js` lines 6783-7009 (proxy logic)
- **Handler:** `forwardGenericApiRequest()` - translates requests
- **SSE:** `/api/event` → OpenCode `/event` (real-time updates)
- **Auth:** `OPENCODE_SERVER_PASSWORD` env var

### Client-Side SDK
- **File:** `packages/ui/src/lib/opencode/client.ts`
- **Lib:** `@opencode-ai/sdk/v2`
- **Base URL:** Default `/api` (proxied), override with `VITE_OPENCODE_URL`
- **Exported:** `opencodeClient` singleton (used by stores)

### Event Streaming (SSE)
- **Hook:** `packages/ui/src/hooks/useEventStream.ts`
- **Events:** `session.*`, `message.*`, `tool.*`, `permission.required`, `todo.updated`
- **Real-time:** State updates via Zustand stores

### Configuration
```bash
# External server
OPENCODE_HOST=http://hostname:4096
OPENCODE_SKIP_START=true
OPENCODE_SERVER_PASSWORD=secret

# Managed spawn
OPENCODE_PORT=4096
OPENCHAMBER_OPENCODE_HOSTNAME=127.0.0.1
OPENCODE_BINARY=/path/to/opencode
OPENCODE_WSL_DISTRO=Ubuntu
```

### Key Files (30-second scan)

**Spawn & Proxy:**
- `packages/web/server/index.js` - Everything OpenCode-related (14K lines)

**OpenCode Module:**
- `packages/web/server/lib/opencode/` - Config, auth, skills, providers

**Client SDK:**
- `packages/ui/src/lib/opencode/client.ts` - SDK wrapper
- `packages/ui/src/hooks/useEventStream.ts` - SSE handler
- `packages/ui/src/stores/useSessionStore.ts` - State + integration

**Architecture:**
- `AGENTS.md` - Read first (mandatory)
- `packages/web/server/lib/opencode/DOCUMENTATION.md` - OpenCode module API

### No Backend Abstraction
- OpenCode is **tightly coupled** (HTTP proxy, SSE, SDK types)
- No pluggable provider interface for swapping AI backends
- Provider management = OpenCode config, not abstraction layer

### Important Constants
- **SSE timeout:** 4 min (LONG_REQUEST_TIMEOUT_MS = 240s)
- **Health check:** 10s timeout
- **Message size:** 50MB limit
- **Default port:** 4096 (if not configured)
