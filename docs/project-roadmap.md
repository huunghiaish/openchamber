# OpenChamber Project Roadmap

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Status:** Active Development

---

## Current State (Q1 2026)

### ✅ Completed

**Core Platform:**
- [x] Web/PWA with Cloudflare tunnel support (quick, managed-remote, managed-local)
- [x] macOS Desktop app (Tauri 2, sidecar server, multi-window)
- [x] VS Code extension (chat sidebar, agent manager, file integration)
- [x] Shared React UI library (506 files, 36+ Zustand stores)

**Feature Set:**
- [x] Branchable chat timeline (/undo, /redo, fork from any turn)
- [x] Multi-agent runs with isolated worktrees
- [x] Voice mode (speech input + read-aloud)
- [x] Plan/Build mode with dedicated view
- [x] Git integration (status, commit, push, pull, rebase, worktrees)
- [x] GitHub integration (OAuth device flow, PR creation, issue context)
- [x] Integrated terminal (Ghostty Web, PTY over WebSocket)
- [x] Diff viewer (Pierre, stacked/inline modes, large file support)
- [x] File browser (inline editing, markdown preview)
- [x] Settings panel (keyboard shortcuts, theme, layout)
- [x] Skills catalog & local skill management
- [x] Custom theme support (4 built-in, unlimited custom via JSON)
- [x] Quota tracking (20+ AI providers)
- [x] Push notifications (Web Push API)

**Infrastructure:**
- [x] Express backend with 100+ API endpoints
- [x] OpenCode subprocess management (auto-spawn or external)
- [x] SSH remote instance support (desktop app)
- [x] Docker deployment (multi-stage, docker-compose)
- [x] systemd service integration
- [x] Auto-update mechanism (desktop + web)
- [x] Security: UI password, tunnel tokens, OAuth, path validation

---

## Roadmap by Quarter

### Q2 2026: Windows & Linux Desktop

**Timeline:** April - June 2026

**Goals:**
- Extend native desktop experience to Windows & Linux
- Maintain feature parity with macOS
- Platform-specific menu integrations

**Tasks:**

1. **Windows (10+) Desktop App**
   - [ ] Port Tauri app to Windows
   - [ ] Update Rust code for Windows-specific APIs
   - [ ] Create Windows installer (.msi or MSIX)
   - [ ] Test on Windows 10 & 11 (x86_64, arm64)
   - [ ] Windows-specific shortcuts & menus

2. **Linux Desktop App**
   - [ ] Port Tauri app to Linux (x86_64, aarch64)
   - [ ] Support common distros (Ubuntu 22.04+, Fedora 38+, Debian 12+)
   - [ ] Create .AppImage + .deb packages
   - [ ] Desktop integration (system menus, notifications)
   - [ ] Wayland + X11 support

3. **Cross-Platform Features**
   - [ ] Platform-agnostic file dialogs
   - [ ] System notifications (all platforms)
   - [ ] Auto-update (Windows/Linux parity with macOS)
   - [ ] SSH key management (Windows)
   - [ ] Winget / Homebrew / apt package managers

**Testing:**
- [ ] CI/CD: Build .msi, .exe, .AppImage, .deb on releases
- [ ] E2E tests on Windows & Linux runners
- [ ] User acceptance testing (beta group)

**Metrics:**
- Desktop app downloads: 10K+ monthly (all platforms)
- Zero critical platform-specific bugs in first month

---

### Q3-Q4 2026: Mobile App & Enhanced Tunneling

**Timeline:** July - December 2026

**Phase 1: Mobile (Q3)**

**Goals:**
- Native iOS/Android app for remote review & management
- Laptop connectivity for developers on-the-go
- Touch-optimized UI

**Tasks:**

1. **iOS App (React Native)**
   - [ ] Create packages/mobile-ios (React Native)
   - [ ] Chat interface (mobile-optimized)
   - [ ] Git operations (view status, view diffs)
   - [ ] Terminal access (read-only initially)
   - [ ] Session list & switching
   - [ ] Touch gestures (swipe, pinch, long-press)
   - [ ] Build & sign for iOS 16.4+
   - [ ] App Store distribution

2. **Android App (React Native)**
   - [ ] Create packages/mobile-android (shared React Native codebase)
   - [ ] Feature parity with iOS
   - [ ] Material Design 3 integration
   - [ ] Build & sign for Android 9+
   - [ ] Google Play Store distribution

3. **Laptop Connectivity**
   - [ ] "Pair with laptop" QR code flow
   - [ ] Local network discovery (mDNS/Bonjour)
   - [ ] WebSocket tunnel for cross-device sync
   - [ ] Session handoff (start on desktop, continue on phone)

4. **Mobile-Specific Features**
   - [ ] Offline message drafts (sync on reconnect)
   - [ ] Battery optimization (lower refresh rates)
   - [ ] Notification routing (chat, tool outputs, errors)
   - [ ] Biometric auth (Face ID, fingerprint)

**Testing:**
- [ ] iPhone 13+ simulator & real devices
- [ ] Android 9+ emulator & real devices
- [ ] Network latency tests (3G, 4G, LTE)
- [ ] Battery consumption profiling

**Metrics:**
- Mobile app downloads: 1K+ in first month
- Positive reviews (4.5+/5 stars)

---

**Phase 2: Enhanced Tunneling (Q3-Q4)**

**Goals:**
- Multiple tunnel provider options
- More enterprise-friendly deployment

**Tasks:**

1. **Additional Tunnel Providers**
   - [ ] ngrok integration (simple, widely used)
   - [ ] WireGuard tunnel mode (decentralized)
   - [ ] Tailscale integration (via API)
   - [ ] Self-hosted tunnel server option (optional deploy)

2. **Tunnel Features**
   - [ ] Tunnel analytics (bandwidth, latency, uptime)
   - [ ] IP whitelisting (for security)
   - [ ] Custom domain management (CNAME delegation)
   - [ ] Tunnel failover (automatic + manual)

3. **UX Improvements**
   - [ ] Tunnel setup wizard (guided)
   - [ ] One-click teardown (revoke all tokens)
   - [ ] Tunnel session history & logs
   - [ ] Per-tunnel rate limiting config

**Testing:**
- [ ] Multi-provider stress tests
- [ ] High-latency scenarios (satellite, VPN)
- [ ] Concurrent tunnel sessions

**Metrics:**
- 50+ tunnel sessions active simultaneously
- <100ms latency for managed tunnels

---

### Q4 2026 - Q1 2027: Multi-Agent Kanban & Plugins

**Timeline:** October 2026 - March 2027

**Phase 1: Multi-Agent Kanban (Q4 2026)**

**Goals:**
- Visual board for orchestrating multi-agent workflows
- Human-in-the-loop decision making
- Parallel task tracking

**Tasks:**

1. **Kanban Board UI**
   - [ ] Create packages/ui/components/kanban (React component)
   - [ ] Columns: Proposed → In Progress → Blocked → Complete
   - [ ] Drag-and-drop card management
   - [ ] Agent status indicators (running, idle, error)
   - [ ] Real-time updates via SSE

2. **Agent Task Management**
   - [ ] Task creation from chat turns
   - [ ] Agent assignment (single or multiple)
   - [ ] Priority & deadline tracking
   - [ ] Worktree association visualization
   - [ ] Dependency graphs (task A blocks task B)

3. **Human-in-the-Loop Controls**
   - [ ] Approval gates (human reviews before execution)
   - [ ] Task pausing (freeze agent midway)
   - [ ] Manual intervention prompts
   - [ ] Conflict resolution UI (when agents disagree)

4. **Integration with Existing Features**
   - [ ] Kanban ↔ Chat message linking
   - [ ] Worktree visualization on board
   - [ ] Git branch per task (optional)
   - [ ] Cost tracking per task

**Testing:**
- [ ] Stress test: 50+ parallel agents
- [ ] Drag-drop performance with 100+ tasks
- [ ] SSE stability with 10 concurrent updates/sec

**Metrics:**
- Users running 3+ parallel agents: >40%
- Average workflow completion time: <30% reduction

---

**Phase 2: Plugin/Extensions Catalog (Q1 2027)**

**Goals:**
- Extensible plugin system for custom tools & integrations
- Built-in plugin marketplace
- Developer-friendly plugin API

**Tasks:**

1. **Plugin Architecture**
   - [ ] Define plugin manifest format (package.json + config)
   - [ ] Plugin discovery system (built-in + remote)
   - [ ] Sandboxing & permission model
   - [ ] Hot-reload support (no restart)

2. **Built-In Plugins**
   - [ ] GitHub integrations (more workflows)
   - [ ] Linear issue tracker
   - [ ] Slack notifications
   - [ ] Custom build command runners

3. **Plugin Marketplace (ClawdHub Integration)**
   - [ ] Plugin listing & discovery UI
   - [ ] One-click install/uninstall
   - [ ] Version management & updates
   - [ ] Plugin ratings & reviews
   - [ ] Developer portal (publish plugins)

4. **Plugin API**
   - [ ] Tool execution hooks
   - [ ] UI component registration
   - [ ] Event subscriptions (chat, git, terminal)
   - [ ] Storage (persistent key-value)
   - [ ] Command palette extensions

5. **Example Plugins**
   - [ ] Jira integration
   - [ ] AWS CodePipeline monitoring
   - [ ] Custom Docker registry support
   - [ ] Datadog APM integration

**Testing:**
- [ ] Plugin isolation (prevent crashing main app)
- [ ] Permission validation (no unauthorized access)
- [ ] Performance: 10+ plugins loaded

**Metrics:**
- Plugin ecosystem: 20+ community plugins
- Plugin adoption: >60% of users enable ≥1 plugin

---

### Future (2027+): Browser & Integrations

**Timeline:** 2027 onwards

**Features in Consideration:**

1. **Built-In Browser**
   - [ ] Preview dev servers directly in UI
   - [ ] Agent integration (click elements, take screenshots)
   - [ ] Network inspector (see API calls)
   - [ ] Console logs in sidebar

2. **Linear Integration**
   - [ ] Create issues from chat
   - [ ] Link issues to sessions
   - [ ] Update issue status from within OpenChamber
   - [ ] Roadmap visualization

3. **More LLM Providers**
   - [ ] Anthropic Claude (already supported)
   - [ ] OpenAI GPT-4 (already supported)
   - [ ] Google Gemini (enhanced)
   - [ ] Llama 2 / LLaMA (local)
   - [ ] Mistral (new)
   - [ ] Custom/self-hosted endpoints

4. **Collaboration Features**
   - [ ] Real-time pair programming (shared sessions)
   - [ ] Comment threads on messages
   - [ ] Session sharing (read-only or edit)
   - [ ] Team workspace management

5. **Advanced Analytics**
   - [ ] Agent performance metrics (accuracy, speed, cost)
   - [ ] Code quality trends (from diffs)
   - [ ] Team velocity tracking
   - [ ] AI spend breakdown by team/project

---

## Dependency Map

```
Core Features (no external deps)
├── Chat interface
├── Message streaming
├── Session management
└── File browser

↓ depends on ↓

Backend Services
├── OpenCode integration
├── Git operations
├── GitHub API
└── Terminal PTY

↓ depends on ↓

Infrastructure
├── Tunnel providers (Cloudflare, ngrok, etc.)
├── Database/persistence (localStorage, sqlite)
└── Auth (OAuth, UI password)

↓ depends on ↓

Deployment Targets
├── Web PWA
├── Desktop (macOS, Windows, Linux)
├── Mobile (iOS, Android)
└── VS Code extension
```

---

## Resource Allocation

### Team Capacity (Estimated)

- **1 Senior Full-Stack** — Architecture, complex features, reviews
- **2 Frontend Engineers** — UI/UX, components, mobile
- **1 Backend Engineer** — API, integrations, DevOps
- **1 DevOps/SRE** — CI/CD, deployment, monitoring (part-time)
- **1 QA/Tester** — E2E tests, cross-platform validation

### Priority Allocation (2026)

| Phase | Effort | Team |
|-------|--------|------|
| **Windows/Linux** | 40% | Full-Stack (2), Backend (0.5) |
| **Mobile** | 30% | Frontend (2), Backend (0.5) |
| **Tunnel Providers** | 15% | Full-Stack (0.5), Backend (0.5) |
| **Kanban** | 10% | Frontend (1) |
| **Maintenance** | 5% | QA, DevOps |

---

## Success Metrics

### Adoption

| Metric | Target | Timeline |
|--------|--------|----------|
| Desktop downloads | 10K+/month | Q2 2026 |
| Web tunnels | 5K active | Q1 2026 |
| Mobile downloads | 1K+/month | Q3 2026 |
| VS Code installs | 2K+ | Q2 2026 |

### Engagement

| Metric | Target | Timeline |
|--------|--------|----------|
| DAU (daily active users) | 1K | Q2 2026 |
| Multi-agent runs | >40% users | Q4 2026 |
| Custom plugins | 20+ ecosystem | Q1 2027 |
| Session duration | >30 min avg | Q2 2026 |

### Quality

| Metric | Target | Timeline |
|--------|--------|----------|
| Crash rate | <0.1% | Continuous |
| E2E test coverage | >80% | Q2 2026 |
| Performance (TTI) | <2s | Q2 2026 |
| WCAG 2.1 AA | 100% | Q3 2026 |

### Retention

| Metric | Target | Timeline |
|--------|--------|----------|
| 30-day retention | >70% | Q2 2026 |
| Churn rate | <5%/month | Q3 2026 |
| NPS score | >50 | Q2 2026 |

---

## Risk Assessment

### High Risk

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Tauri cross-platform bugs | High | Early testing, community feedback, fallback builds |
| OpenCode API breaking changes | High | Version pinning, adapter pattern, upstream coordination |
| Mobile app store delays | Medium | Beta via TestFlight/Play Beta early, grow web PWA first |

### Medium Risk

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Cloudflare tunnel rate limits | Medium | Support multiple providers, implement fallback |
| Large team churn | Medium | Good documentation, knowledge sharing, mentorship |
| TypeScript strict mode adoption | Low | Gradual enforcement, refactoring sprints |

---

## Technical Debt Tracking

### Known Debt Items

1. **Consolidate Zustand Stores** (36→12 stores)
   - Effort: 2 sprints
   - Impact: 30% faster state initialization
   - Priority: Q2 2026

2. **Refactor server/index.js** (14,413 lines)
   - Break into lib/ modules (opencode, git, github, terminal, etc.)
   - Effort: 4 sprints
   - Impact: 50% faster feature development
   - Priority: Q3 2026

3. **Extract Terminal I/O Logic**
   - Create reusable module (node-pty abstraction)
   - Effort: 1 sprint
   - Impact: Mobile & plugin support
   - Priority: Q3 2026

4. **Add E2E Tests**
   - Playwright/Cypress setup
   - Critical user journeys (20+ scenarios)
   - Effort: 2 sprints
   - Impact: 90% reduction in regressions
   - Priority: Q2 2026

---

## Community & Contribution

### Contribution Guidelines

- OSS-friendly licensing (MIT already)
- Clear CONTRIBUTING.md (in progress)
- Good first issues tagged for newcomers
- Weekly community standup (optional)

### External Partners

1. **OpenCode Team** — API stability, feature coordination
2. **Tauri Project** — Cross-platform support, bug reports
3. **Cloudflare** — Tunnel provider partnership
4. **GitHub** — OAuth, API improvements

### Community Channels

- Discord server (link in README)
- GitHub Discussions
- Product feedback (Hacker News, Reddit)

---

## Release Schedule

### Version Numbering

**Semantic Versioning: MAJOR.MINOR.PATCH**

- **MAJOR:** Breaking changes, major platform support (Windows/Linux, mobile)
- **MINOR:** New features, enhancements (tunnel providers, kanban)
- **PATCH:** Bug fixes, security updates

### Release Cadence

- **Patch releases:** Weekly (fixes, small improvements)
- **Minor releases:** Bi-weekly (features)
- **Major releases:** Quarterly (platform updates)

**Latest Version (as of 2026-03-24):** v1.9.1

**Next Planned Releases:**
- v1.10.0 (Q2 2026) — Windows/Linux desktop support
- v2.0.0 (Q3 2026) — Mobile app launch
- v2.5.0 (Q4 2026) — Kanban board + multi-agent orchestration

---

## Documentation Roadmap

| Doc | Status | Priority | Owner |
|-----|--------|----------|-------|
| **project-overview-pdr.md** | ✅ Done | Essential | Tech Lead |
| **codebase-summary.md** | ✅ Done | Essential | Tech Lead |
| **code-standards.md** | ✅ Done | Essential | Tech Lead |
| **system-architecture.md** | ✅ Done | Essential | Tech Lead |
| **deployment-guide.md** | ✅ Done | Essential | Tech Lead |
| **design-guidelines.md** | ✅ Done | Essential | Design Lead |
| **project-roadmap.md** | ✅ Done | Essential | Product Manager |
| **CUSTOM_THEMES.md** | ✅ Done | Important | Tech Lead |
| **API Reference** | Pending | Important | Backend Engineer |
| **Plugin Development Guide** | Pending | Medium | Frontend Engineer |
| **Mobile Development Guide** | Pending | Medium | Mobile Engineer |
| **Troubleshooting Guide** | Pending | Low | Support |

---

## How to Use This Roadmap

### For Contributors

1. **Pick a task** from a phase that interests you
2. **Check dependencies** — make sure blockers are resolved
3. **Open an issue** with `[RFC]` tag for discussion
4. **Submit PR** with reference to issue
5. **Get reviewed** by team lead

### For Users

1. **Vote on features** via GitHub Reactions (👍)
2. **Report bugs** with detailed reproduction steps
3. **Join Discord** for announcements
4. **Test beta features** and provide feedback

### For Product Managers

1. **Track progress** via GitHub Projects board
2. **Adjust timeline** based on user feedback & market conditions
3. **Update quarterly** (every 3 months)
4. **Sync with team** weekly on blockers

---

**Maintained by:** OpenChamber Team | **License:** MIT

**Last Review:** 2026-03-24 | **Next Review:** 2026-06-24 (Q2 milestone)
