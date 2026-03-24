# OpenChamber: Project Overview & Product Requirements

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Status:** Active Development

---

## What is OpenChamber?

OpenChamber is a rich, cross-platform GUI interface for **OpenCode** — an AI-powered coding assistant. It provides visual workflows for code review, agent management, terminal access, and git/GitHub operations across desktop, web, and mobile platforms.

**Core Mission:** Enable developers to leverage OpenCode through an intuitive, feature-rich interface while maintaining full control and visibility of AI-assisted workflows.

---

## Why OpenChamber Exists

### Problem Statement
OpenCode's core TUI limits user reach and workflow styles:
- **Power users** prefer visual interfaces for code review and multi-agent orchestration
- **Remote teams** need browser-based, tunnel-accessible access from any device
- **Mobile developers** want to review diffs and manage sessions on tablets/phones
- **GUI-first developers** need a familiar desktop application (macOS initially)

### Solution
OpenChamber provides:
1. **Desktop App** (macOS) — native experience with offline sidecar server
2. **Web PWA** — instant access via browser, tunnel support, mobile-optimized
3. **VS Code Extension** — editor-native workflow without context switching
4. All three share a **unified React UI library** ensuring consistency

---

## Target Users

| User Profile | Primary Entry | Key Features |
|---|---|---|
| **Desktop Developer** | macOS app | Multi-window, native menus, fast startup |
| **Remote Collaborator** | Web PWA + Cloudflare tunnel | QR onboarding, background notifications |
| **VS Code User** | Extension | Side-by-side chat, file integration, theme sync |
| **Mobile Reviewer** | PWA on tablet | Touch-optimized UI, responsive layout |
| **Self-Hosted Operator** | Docker / systemd | Full control, no cloud dependency |

---

## Functional Requirements

### Chat & Reasoning
- **Branchable Chat Timeline** — `/undo`, `/redo`, fork from any turn without losing history
- **Message Streaming** — Part-based architecture with proper token handling
- **Voice Mode** — Speech input + read-aloud for hands-free workflows
- **Multi-Agent Runs** — Single prompt spawns parallel agents with isolated worktrees
- **Inline Comments** — Draft feedback on diffs/files, send back to agent

### Code & Git
- **Git Sidebar** — Status, staging, commits, push/pull, branch/rebase/merge, worktrees
- **Diff Viewer** — Stacked/inline modes, Pierre syntax highlighting, large file support
- **GitHub Integration** — OAuth device flow, PR creation/merge, issue context
- **Terminal** — Per-directory sessions, heavy output stability, Ghostty Web UI
- **File Browser** — Inline editing, markdown preview, file-type icons

### Productivity & Control
- **Plan/Build Mode** — Dedicated view for drafting & iterating implementation steps
- **Context Visibility** — Token/cost breakdowns, raw message inspection, activity logs
- **Skills Management** — Built-in catalog + local skill uploads
- **Quota Tracking** — Real-time provider monitoring (20+ AI services)
- **Settings** — Keyboard shortcuts, font size, theme, layout controls

### Web-Specific (PWA)
- **Cloudflare Tunnels** — quick, managed-remote, managed-local modes with secure tokens
- **One-Scan Onboarding** — QR code + password-protected setup
- **Background Notifications** — Web Push API integration
- **Self-Update** — In-app update + restart while preserving config
- **Mobile-First** — Touch controls, keyboard-safe, attachment-friendly

### Desktop-Specific (macOS)
- **Native Menus** — App actions, deep linking, window management
- **Multi-Window** — Parallel project workflows with persistent geometry
- **SSH Remotes** — Connect to remote OpenChamber instances via SSH tunnel + port forward
- **Project Actions** — Dev server spawn, SSH port forwarding, open remote URLs locally
- **Auto-Update** — Ed25519-signed GitHub releases via Tauri

### VS Code Extension
- **Editor-Native** — Open files from tool output, sessions beside code
- **Agent Manager** — Panel for parallel multi-model runs
- **Right-Click Actions** — Add context, explain, improve code in-place
- **Theme Sync** — Map VS Code theme to OpenChamber theme
- **Responsive Layout** — Adapt to editor width, proper focus management

---

## Non-Functional Requirements

### Performance
- **Bundle Size:** <1.2 MB per chunk (Vue.js constraints)
- **Time-to-Interactive:** <2s on 4G, <500ms on LAN
- **Terminal Output:** Stable at 1000+ lines/sec
- **Diff Rendering:** Handle 10K+ line diffs without blocking
- **Memory:** <300 MB baseline (web), <500 MB (desktop)

### Reliability
- **Uptime:** Graceful degradation if OpenCode unavailable
- **Offline Fallback:** PWA works in degraded mode; Tauri desktop fully local
- **State Persistence:** Settings, sessions, themes survive restarts
- **Error Recovery:** Auto-reconnect on network loss, no session loss

### Security
- **UI Password:** Optional, defaults disabled
- **Tunnel Tokens:** One-time use, per-session
- **Local Binding:** 127.0.0.1 by default; explicit `--host` for LAN
- **Path Validation:** No directory traversal; symlink detection
- **OAuth:** Device flow only; tokens stored mode 600
- **SSH:** User-controlled explicit configuration, no auto-discovery

### Accessibility
- **WCAG 2.1 AA** — Keyboard navigation, screen reader support
- **Mobile Friendly** — Touch targets 44px+, no horizontal scroll
- **High Contrast** — Multiple theme options including high-contrast variant
- **Respects Preferences** — prefers-color-scheme, prefers-reduced-motion

### Compatibility
- **Browsers:** Chrome 120+, Safari 16+, Firefox 120+
- **Desktop:** macOS 13.0+ (arm64 + x86_64)
- **Platforms:** Web (all OS), PWA (iOS 16.4+, Android 9+), Extension (VS Code 1.85+)

---

## Key Architectural Decisions

### Monorepo (Bun Workspaces)
- **Reasoning:** Shared UI across desktop/web/extension; unified type system
- **Trade-off:** Simpler dependency management vs. larger monorepo

### React 19 + TypeScript (Strict)
- **Reasoning:** Modern SSR-ready, type safety, component composition
- **Trade-off:** Steeper onboarding vs. long-term maintainability

### Zustand for State (36+ Stores)
- **Reasoning:** Lightweight, persistent stores, devtools support
- **Trade-off:** No async middleware; manual coordination needed

### Part-Based Message Streaming
- **Reasoning:** Accurate token counting, proper turn structure, agent isolation
- **Trade-off:** Client complexity; requires careful buffer management

### Tauri 2 + Sidecar
- **Reasoning:** Native macOS UX; sidecar avoids Electron overhead
- **Trade-off:** Maintenance burden; Rust expertise required

### Vite + PWA Plugin
- **Reasoning:** Fast dev loop, code splitting, iOS-compatible SW
- **Trade-off:** Custom SW needed; precaching adds complexity

---

## Product Roadmap (Next 12 Months)

### Phase 1: Windows/Linux Desktop (Q2 2026)
- Port Tauri app to additional platforms
- Platform-specific menus & integrations
- Binary distribution via GitHub releases

### Phase 2: Mobile App (Q3-Q4 2026)
- Standalone iOS/Android app (React Native)
- Laptop tunnel connectivity
- Touch-optimized chat/terminal

### Phase 3: Tunneling Improvements (Q2-Q3 2026)
- ngrok integration
- Wireguard tunnel mode
- Self-hosted tunnel server option

### Phase 4: Multi-Agent Kanban (Q4 2026)
- Visual board for agent task orchestration
- Real-time status, human-in-loop controls
- Worktree visualization

### Phase 5: Extensions & Integrations (2027)
- OpenCode custom plugins/tools catalog
- Linear integration (issues, projects)
- Built-in browser for dev server preview
- Additional MCP protocol servers

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Desktop Installation | 5K+ monthly | GitHub releases download stats |
| Web Tunnel Sessions | 10K+ monthly | Server telemetry |
| VS Code Extension | 2K+ installs | Marketplace stats |
| Time-to-First-Message | <2s | Network + render profiling |
| Crash Rate | <0.1% | Client error telemetry |
| User Retention (30d) | >70% | Session tracking |
| Feature Satisfaction | >4.2/5 | In-app feedback surveys |

---

## Package Breakdown

### packages/ui (Shared UI Library)
- **506 source files** (React, TypeScript, Tailwind CSS 4)
- **36+ Zustand stores** for global state
- **50+ custom components** + Radix UI primitives
- **4 built-in themes** (Flexoki Light/Dark, Fields of the Shire Light/Dark)
- **CodeMirror 6** for code editing, **Ghostty Web** for terminal
- **Entry point:** @openchamber/ui/main (imported by web & extension)

### packages/web (Web Server + PWA)
- **14,413-line Express server** (100+ endpoints)
- **React SPA frontend** (Vite build)
- **CLI:** `openchamber` command (serve, tunnel, logs, update, stop)
- **Services:** OpenCode proxy, Cloudflare tunnels, Git, GitHub OAuth, PTY terminal
- **Entry point:** bin/cli.js (Node.js 20+ or Bun)

### packages/desktop (macOS App)
- **Tauri 2.9.4** framework (Rust 2021)
- **~3400 lines** main.rs (window, menu, IPC)
- **~2970 lines** remote_ssh.rs (SSH tunnel management)
- **23 IPC commands** for native integration
- **Sidecar:** Compiled OpenCode server (via Bun compile)
- **Output:** .app bundle + DMG installer

### packages/vscode (VS Code Extension)
- **22 TypeScript modules** (647-line extension.ts entry)
- **Bridge protocol** (RPC messages to webview)
- **Webview:** React + @openchamber/ui (Vite-built)
- **Features:** Chat sidebar, agent manager, session editor, theme sync
- **Entry point:** extension.ts (loaded by VS Code)

---

## Dependencies Highlights

| Dependency | Version | Role |
|---|---|---|
| React | 19.1.1 | UI framework (all packages) |
| TypeScript | 5.8.3 | Type checking |
| Tailwind CSS | 4.0 | Styling (design tokens via CSS variables) |
| Zustand | 5.0.8 | State management (36+ stores) |
| Radix UI | Latest | Accessible component primitives |
| CodeMirror 6 | Latest | Code editor + syntax highlighting |
| Express | 5.1.0 | Web server |
| Tauri | 2.9.4 | Desktop framework |
| Simple-git | 3.28.0 | Git operations |
| @octokit/rest | 22.0.1 | GitHub API |

---

## Development Standards

- **TypeScript:** Strict mode enforced; path aliases for imports
- **Styling:** Tailwind CSS v4 + design tokens (no arbitrary values in components)
- **Components:** Functional + hooks; CVA variants for style composition
- **Naming:** kebab-case files, camelCase exports
- **Linting:** ESLint 9 + typescript-eslint + react-hooks
- **Git:** Conventional commits (feat:, fix:, docs:, refactor:, test:)
- **Testing:** Unit tests required for lib/ code; integration tests for API
- **Security:** No plaintext secrets; OAuth tokens in ~/.opencode/auth.json (mode 600)

---

## Known Constraints & Future Work

### Current Limitations
1. **Desktop macOS-only** (Windows/Linux planned Q2 2026)
2. **Single tunnel per instance** (multiple ports require multiple instances)
3. **Vite chunk limit** at 1.2 MB (large dependency bundles flagged)
4. **PWA offline** is minimal (focused on push notifications)
5. **File uploads** capped at 50 MB

### Technical Debt
- Consolidate 36+ Zustand stores (consider reducing to 10-12 core stores)
- Move message part rendering to micro-components
- Extract terminal I/O logic to separate module
- Add end-to-end tests for tunnel + SSH flows

### Future Enhancements
- React 19 experimental features (suspense, transitions)
- Code Splitting: lazy-load views/stores on demand
- Offline Sync: local message drafts, conflict resolution
- Plugin API: allow third-party UI extensions
- Localization: i18n support for multiple languages

---

## Reference Links

- **README:** /Users/nghia/Projects/openchamber/README.md
- **Code Standards:** /Users/nghia/Projects/openchamber/docs/code-standards.md
- **System Architecture:** /Users/nghia/Projects/openchamber/docs/system-architecture.md
- **Codebase Summary:** /Users/nghia/Projects/openchamber/docs/codebase-summary.md
- **Deployment Guide:** /Users/nghia/Projects/openchamber/docs/deployment-guide.md
- **Design Guidelines:** /Users/nghia/Projects/openchamber/docs/design-guidelines.md
- **Custom Themes:** /Users/nghia/Projects/openchamber/docs/CUSTOM_THEMES.md

---

**Maintained by:** OpenChamber maintainers | **License:** MIT
