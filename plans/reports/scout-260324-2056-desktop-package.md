# Scout Report: OpenChamber Desktop Package (Tauri)

**Date:** March 24, 2026 | **Focus:** packages/desktop/ | **Status:** Complete

---

## Tech Stack Overview

### Framework & Tooling
- **Tauri 2.9.4** (Rust+WebView native framework)
- **Rust 2021 edition** (backend/native functionality)
- **TypeScript 5.8.3** (type definitions)
- **Bun** (build tool, sidecar compilation)
- **Vite** (dev server HMR in dev mode)

### Rust Dependencies (Key)
| Dep | Version | Purpose |
|-----|---------|---------|
| tauri | 2.9.4 | Core framework, macOS private API |
| tauri-plugin-shell | 2.3.3 | Shell command execution, open URLs |
| tauri-plugin-dialog | 2.4.2 | File/folder selection dialogs |
| tauri-plugin-notification | 2.3.3 | Native notifications |
| tauri-plugin-updater | 2 | Auto-update mechanism |
| tauri-plugin-log | 2.7.1 | Logging to stdout + webview |
| window-vibrancy | 0.7.1 | macOS vibrancy effects |
| tokio | 1.38 | Async runtime (multi-threaded) |
| reqwest | 0.12.4 | HTTP client (for health checks, probes) |
| serde/serde_json | 1.0.x | Serialization |

### Platform Support
- **Primary:** macOS only (minimal platform-specific code)
- **Entitlements:** entitlements.plist configured for security sandbox
- **Minimum macOS:** 13.0 (Big Sur)
- **Architectures:** aarch64-apple-darwin (M1/M2+), x86_64-apple-darwin (Intel)

---

## Directory Structure

```
packages/desktop/
├── package.json                    # npm metadata, Tauri CLI v2, TypeScript
├── README.md                       # Feature overview, install instructions
├── noop-dist/                      # Placeholder for production builds
├── public/                         # Static assets
│   └── ibm-plex-mono*.woff2       # System fonts (monospace)
├── scripts/
│   ├── build-sidecar.mjs          # Compiles OpenCode server to binary sidecar
│   ├── dev-web-server.mjs         # Starts API server (port 3901) + Vite HMR
│   ├── opencode-cli.mjs           # CLI utilities (unused in desktop)
│   └── desktop-dev.mjs            # Orchestrates dev environment
├── src-tauri/                      # Rust backend
│   ├── Cargo.toml                 # Rust dependencies + features
│   ├── Cargo.lock                 # Dependency lock
│   ├── src/
│   │   ├── main.rs                # ~3400 lines: window mgmt, IPC commands, app lifecycle
│   │   └── remote_ssh.rs          # ~2970 lines: SSH tunnel, remote instance mgmt
│   ├── tauri.conf.json            # Main Tauri config (windows, build, plugins)
│   ├── tauri.dev.conf.json        # Dev-specific overrides (dev icons)
│   ├── build.rs                   # Build script (minimal)
│   ├── Info.plist                 # macOS app metadata
│   ├── entitlements.plist         # macOS sandbox permissions
│   ├── capabilities/
│   │   └── default.json           # Tauri v2 capability (permissions, windows, urls)
│   ├── icons/
│   │   ├── icon.icns              # App icon (macOS)
│   │   ├── icon.png               # Fallback icon
│   │   ├── dev-icon.icns          # Dev build icon
│   │   └── dev-icon.png
│   ├── resources/
│   │   └── web-dist/              # Bundled web UI (built by Vite, copied by build-sidecar.mjs)
│   └── sidecars/                  # Compiled OpenCode server binary
│       └── openchamber-server-{target-triple} (built at compile time)
```

---

## Native Features & IPC

### Window Management
- **Single + Multi-Window:** Startup creates "main" window, additional windows labeled "main-N"
- **Window Counter:** Global atomic counter tracks unique labels
- **Overlay Title Bar (macOS):** Custom chrome with traffic light at (17, 26)
- **Transparency:** True transparency enabled on macOS
- **Theme Integration:** Dark/light mode with vibrancy effects via window-vibrancy crate
- **Geometry Persistence:** Saves/restores window size, position, max/fullscreen state
- **Focus Tracking:** Tracks focused window per application lifecycle
- **Background Throttling:** Disabled (performance-critical for chat/terminal)

### Menu System (macOS-Only)
Dynamic menu built at app startup. Keyboard shortcuts + menu items dispatch to UI via:
1. Tauri event emission: `app.emit("openchamber:menu-action", action)`
2. Window eval (focused window preferred): `window.dispatchEvent(new CustomEvent(...))`

**Menu Actions:**
- New Window, Check For Updates, Report Bug, Request Feature, Join Discord
- About, Settings, Command Palette, New Session, Worktree Creator, Change Workspace
- Git/Diff/Files/Terminal tabs
- Theme (Light/Dark/System), Toggle Sidebar, Memory Debug
- Help, Download Logs, Clear Cache

### IPC Commands (Tauri 2 invoke_handler)

#### Window & App Control
- `desktop_new_window()` → Create new window at default URL
- `desktop_new_window_at_url(url)` → Create window pointing to specific host
- `desktop_set_window_theme(theme)` → Apply light/dark/system theme
- `desktop_restart()` → Restart entire app

#### File Operations
- `desktop_open_path(path, app?)` → Open file/folder (optional app override)
- `desktop_open_in_app(project_path, app_id)` → Open in TextEdit, Terminal, Finder, etc.
- `desktop_read_file(path)` → Read + base64 encode file (drag-drop attachments)

#### App Discovery (macOS)
- `desktop_get_installed_apps(apps)` → Check installed app bundles, fetch icons, cache results
- `desktop_filter_installed_apps(apps)` → Fast filter against cache
- `desktop_fetch_app_icons(apps)` → Extract icons from .app bundles → data URLs
- **Caching:** 24-hour TTL on installed apps list

#### Host & Server Management
- `desktop_hosts_get()` → Load remote hosts config from disk
- `desktop_hosts_set(config)` → Save remote hosts config
- `desktop_host_probe(url)` → Health check endpoint (soft 5s + hard 30s timeouts)
- **Config Storage:** `~/.openchamber/desktop-hosts.json`

#### System Notifications
- `desktop_notify(payload)` → Show native macOS notification (title, body, tag, icon)

#### Software Updates
- `desktop_check_for_updates()` → Poll GitHub releases, return DesktopUpdateInfo
- `desktop_download_and_install_update()` → Download + install, restart on completion
- **Updater Plugin:** Configured with GitHub release endpoint + public key

#### SSH Remote Instances (remote_ssh.rs)
- `desktop_ssh_instances_get()` → Load SSH instances config
- `desktop_ssh_instances_set(config)` → Save SSH instances config
- `desktop_ssh_import_hosts()` → Scan `~/.ssh/config` for candidates
- `desktop_ssh_connect(instance_id)` → Establish SSH tunnel, spawn OpenCode server
- `desktop_ssh_disconnect(instance_id)` → Terminate SSH tunnel + cleanup
- `desktop_ssh_status(instance_id)` → Get current connection state
- `desktop_ssh_logs(instance_id)` → Fetch last 1200 lines of logs
- `desktop_ssh_logs_clear(instance_id)` → Clear logs for instance

---

## Sidecar & Runtime Architecture

### Sidecar: OpenCode Server
**What:** Compiled Node.js binary (Bun compile target) bundled with app.

**Build Process:**
1. `build-sidecar.mjs`: 
   - Runs `bun build --compile` on `packages/web/server/index.js`
   - Targets platform-specific triple (aarch64-apple-darwin / x86_64-apple-darwin)
   - Outputs: `src-tauri/sidecars/openchamber-server-{triple}`
   - Chmod 755 (executable)

2. `tauri.conf.json`:
   ```json
   "bundle": {
     "externalBin": ["sidecars/openchamber-server"],
     "resources": ["resources/web-dist/**/*"]
   }
   ```
   → Sidecar + web assets bundled into .app

**Runtime (app startup):**
```
create_startup_window() [show splash]
  ↓
spawn_local_server() [async]:
  - In debug: wait for dev server (port 3901) or spawn sidecar
  - In release: spawn sidecar with random port
  - Environment: OPENCHAMBER_HOST=127.0.0.1, OPENCHAMBER_DIST_DIR=resources/web-dist
  - Health check loop (reqwest, 5s intervals)
  ↓
activate_main_window() [load UI]:
  - Select URL: env override → default host → local
  - Probe remote hosts (soft 5s, hard 30s)
  - Navigate webview to chosen URL
```

### UI Loading
- **Production:** `src-tauri/resources/web-dist/` (bundled)
- **Dev:** Vite HMR server (http://127.0.0.1:5173) with fallback to sidecar
- **Init Script:** Injected on page load via `initialization_script(...)`, sets up:
  - Theme override (light/dark/system from settings)
  - Local origin header (for CORS isolation)
  - Window event listeners for menu actions

---

## Desktop-Specific Integrations

### macOS Vibrancy
- Applied via `window-vibrancy` crate (NSVisualEffectMaterial)
- Automatically applied on window creation
- Disabled on non-macOS platforms

### Deep Link Handling
- Custom scheme: `openchamber://` (not visible in code, likely handled via Info.plist)
- Supports launching specific workspaces, remotes

### Local Network Security
- CSP: `null` (disabled for full functionality)
- Localhost exception in macOS bundle entitlements
- Allow list: http://127.0.0.1:*/*, http://localhost:*/*, https://*//*

### Settings Persistence
**Stored at:** `~/.openchamber/` (computed via settings_file_path())

| File | Purpose |
|------|---------|
| settings.json | Theme mode, window geometry, user preferences |
| desktop-hosts.json | Remote host list, default host selection |
| desktop-local-port.txt | Last allocated sidecar port (for restart recovery) |

---

## Build & Distribution

### Development
```bash
bun run tauri:dev --features devtools
```
- Starts dev-web-server (sidecar + Vite)
- Launches Tauri dev window
- Hot reload enabled
- Devtools available

### Production Build
```bash
bun run tauri:build
```
- Calls `beforeBuildCommand`: build-sidecar.mjs (compile OpenCode server)
- Calls Tauri build pipeline:
  - Compile Rust → src-tauri/target/release/openchamber-desktop
  - Bundle .app: app icon, sidecar binary, web-dist, Info.plist, entitlements
  - Create DMG (macOS installer) with custom layout
  - Generate updater artifacts (latest.json, signature)

### Auto-Update
- **Endpoints:** GitHub releases (https://github.com/btriapitsyn/openchamber/releases/latest/download/latest.json)
- **Signing:** Ed25519 public key in tauri.conf.json
- **Artifacts:** Signature + binary hosted on GitHub

### Code Signing & Notarization
- **signingIdentity:** null (set at build time or via env)
- **entitlements:** entitlements.plist (local network, file access)
- macOS 13.0+ minimum

---

## Key Behaviors

### On App Launch
1. Create splash window (white background, loading state)
2. Spawn OpenCode server (sidecar) or connect to dev server
3. Determine initial URL:
   - OPENCHAMBER_SERVER_URL env var (highest priority)
   - Config file default host
   - Fallback: local
4. Probe remote URL for health (with retries)
5. Navigate main window to selected URL
6. Show window + request focus

### On Window Closed
- Persist geometry + theme to disk (debounced 500ms)
- Track focus state
- If last window closed: kill sidecar, exit app

### On Menu Event
1. Parse menu item ID
2. Emit Tauri event + window custom event
3. Frontend listens & dispatches (e.g., "new-session", "settings")

### SSH Remote Connection Flow
1. User selects SSH instance
2. `desktop_ssh_connect()` starts tunnel over SSH
3. Spawns OpenCode server on remote
4. Establishes local port forward (127.0.0.1:XXXX → remote:PORT)
5. UI navigates to http://127.0.0.1:XXXX
6. Real-time logs streamed to UI
7. `desktop_ssh_disconnect()` cleans up tunnel + process

---

## Capabilities & Permissions (Tauri v2)

**File:** `capabilities/default.json`

### Windows
- Allow: "main", "main-*" (dynamic multi-window)

### Permissions
| Scope | Actions |
|-------|---------|
| core:window | close, set-title, set-size, set-position, start-dragging |
| core:webview | close, manage |
| shell | open URLs, execute shell commands |
| dialog | file/folder selection, message boxes |
| notification | native notifications, permission checks |
| updater | check updates, download+install |

### Remote URLs (CORS Bypass)
- http://127.0.0.1:*/* (sidecar, dev server)
- http://localhost:*/* (fallback)
- http://* (any HTTP remote, used for SSH tunnels)
- https://* (secure remotes)

---

## Command Patterns & Entry Points

### Main Entry: fn main() at line 2931
- Plugin init: shell, dialog, notification, updater, log
- Menu setup (macOS)
- Window lifecycle handlers
- **invoke_handler** registers all IPC commands at once
- **setup** closure: async initialization, server spawn, window nav
- **run loop**: listens for system events (quit, app focus)

### Command Execution Flow
```
Frontend JS: invoke("command_name", { arg1, arg2 })
  ↓
Tauri deserializes JSON → Rust struct
  ↓
Rust function (async or sync) executes
  ↓
Returns Result<T, String> → JSON response
  ↓
Frontend promise resolves/rejects
```

### State Management
Global state managed via Tauri's State system:
- `SidecarState` → URL of running sidecar
- `DesktopUiInjectionState` → Init script + local origin
- `WindowFocusState` → Focused window tracking
- `WindowGeometryDebounceState` → Geometry persist timer
- `DesktopSshManagerState` → SSH tunnels + logs
- `PendingUpdate` → Updater download state

---

## Notes & Key Findings

### Build Artifacts
- Sidecar compiled at build time to architecture-specific binary
- Web UI bundled separately from Rust (can update independently if needed)
- DMG created for distribution

### Security Model
- Sandbox via entitlements.plist + CSP: null
- Local-only sidecar (127.0.0.1, never 0.0.0.0)
- SSH remotes require user to explicitly add/configure
- File access restricted to workspace + user directories

### Performance Considerations
- Background throttling disabled (needed for real-time chat/terminal)
- Debounced geometry saves (500ms)
- Health checks use reasonable timeouts (5s soft, 30s hard)
- Async spawn for long-running operations (updates, SSH)

### SSH Implementation Highlights
- Multi-instance support with persistent configuration
- Real-time log streaming to UI (max 1200 lines per instance)
- Automatic port forwarding (local → remote)
- Connection status tracking + health probing
- SSH config auto-import from ~/.ssh/config

---

## Files Referenced

**Absolute Paths:**
- `/Users/nghia/Projects/openchamber/packages/desktop/package.json`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/tauri.conf.json`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/Cargo.toml`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/src/main.rs` (3400+ lines)
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/src/remote_ssh.rs` (2970 lines)
- `/Users/nghia/Projects/openchamber/packages/desktop/scripts/build-sidecar.mjs`
- `/Users/nghia/Projects/openchamber/packages/desktop/scripts/dev-web-server.mjs`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/capabilities/default.json`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/Info.plist`
- `/Users/nghia/Projects/openchamber/packages/desktop/src-tauri/entitlements.plist`

