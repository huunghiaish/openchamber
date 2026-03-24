# OpenChamber UI Package Scout Report

**Date:** 2026-03-24  
**Scope:** `/packages/ui/` (Svelte-based React UI component library)  
**Files Analyzed:** 506 source files (.ts/.tsx)

---

## 1. Tech Stack & Dependencies

### Core Framework
- **React:** 19.1.1 (RSC-ready with Server Components hooks)
- **TypeScript:** 5.8.3
- **Vite:** 7.1.2 (build tooling)
- **Tailwind CSS:** 4.0.0 (with PostCSS plugin)

### State Management
- **Zustand:** 5.0.8 (lightweight, immutable store library)
  - 36+ store files using devtools + persist middleware
  - Global state for sessions, messages, config, UI, permissions, etc.

### Component Libraries
- **Radix UI:** Multiple components (dialog, dropdown, select, scroll-area, tooltip, etc.)
- **Remix Icon:** 4.7.0 (icon library)
- **Sonner:** 2.0.7 (toast notifications)
- **Motion:** 12.23.24 (animation library by Framer)
- **dnd-kit:** 6.3.1 + sortable (drag-and-drop)

### Code Editor & Syntax Highlighting
- **CodeMirror 6:** Full ecosystem
  - Languages: JavaScript, TypeScript, Python, Rust, Go, SQL, C++, HTML, CSS, JSON, YAML, XML, Markdown, Elixir, etc.
  - Features: Autocomplete, search, lint, state management
- **Prism.js:** 1.30.0 (legacy fallback syntax highlighting)
- **React Syntax Highlighter:** 15.6.6 (component wrapper)

### Git & Version Control
- **Simple-git:** 3.28.0 (Git operations)
- **Pierre Diffs:** 1.1.0-beta.13 (unified diff rendering)

### Terminal & Shell
- **Ghostty Web:** 0.4.0 (terminal emulator)
- **xterm.js:** (likely dependency of Ghostty)

### Voice & Audio
- **TTS/Voice Services:** Custom implementations with streaming support
- **WebRTC:** Implicit for voice features

### Markdown & Documentation
- **Streamdown:** 2.2.0 (markdown rendering)
- **Beautiful Mermaid:** 1.1.3 (Mermaid diagram rendering)
- **html-to-image:** 1.11.13 (export capabilities)

### Utilities
- **Zod:** 4.3.6 (runtime validation)
- **clsx:** 2.1.1 (className utilities)
- **tailwind-merge:** 3.3.1 (Tailwind class merging)
- **cmdk:** 1.1.1 (command palette)
- **Fuse.js:** 7.1.0 (fuzzy search)
- **yaml:** 2.8.1 (YAML parsing)
- **qrcode:** 1.5.4 (QR code generation)

### OpenCode SDK
- **@opencode-ai/sdk:** 1.3.0 (core AI SDK integration)

### Desktop/Platform Support
- **Tauri:** 2.9.0 (desktop app framework)
- **Express:** 5.1.0 (dev server for proxy, CORS)
- **next-themes:** 0.4.6 (theme provider)

---

## 2. Directory Structure

```
packages/ui/src/
├── main.tsx                          # Entry point (legacy UI)
├── App.tsx                           # Root app component
├── index.css                         # Global styles (1385 lines)
├── App.css                           # App-specific styles
├── vite-env.d.ts                     # Vite type defs
│
├── styles/
│   ├── design-system.css             # Design tokens & custom properties
│   ├── typography.css                # Font & text styles
│   ├── mobile.css                    # Responsive mobile styles
│   ├── fireworks.css                 # Celebration animation styles
│   └── fonts.ts                      # Font loading & configuration
│
├── components/                       # UI components (organized by feature)
│   ├── ui/                           # Base UI components (49 files)
│   │   ├── button.tsx                # Button variants (CVA)
│   │   ├── dialog.tsx                # Modal dialogs
│   │   ├── dropdown-menu.tsx         # Menu components
│   │   ├── select.tsx                # Select dropdown
│   │   ├── tooltip.tsx               # Tooltip wrapper
│   │   ├── sonner.tsx                # Toast config
│   │   ├── scroll-area.tsx           # Custom scrollbars
│   │   ├── command.tsx               # Command palette base
│   │   ├── CommandPalette.tsx        # App-level command palette
│   │   ├── CodeMirrorEditor.tsx      # Embedded code editor
│   │   ├── OverlayScrollbar.tsx      # Custom scrollbar overlay
│   │   ├── ScrollShadow.tsx          # Scroll fade shadows
│   │   ├── text.tsx                  # Text element variants
│   │   ├── textarea.tsx              # Textarea component
│   │   ├── HelpDialog.tsx            # Help/shortcuts dialog
│   │   ├── UpdateDialog.tsx          # App update notifications
│   │   ├── AboutDialog.tsx           # About dialog
│   │   ├── ErrorBoundary.tsx         # Error boundary
│   │   ├── MemoryDebugPanel.tsx      # Debug utilities
│   │   └── ... (more primitives)
│   │
│   ├── layout/                       # Main layout containers (13 files)
│   │   ├── MainLayout.tsx            # Desktop three-pane layout
│   │   ├── VSCodeLayout.tsx          # VS Code extension layout (simplified)
│   │   ├── Header.tsx                # Top toolbar/header
│   │   ├── Sidebar.tsx               # Left sidebar
│   │   ├── RightSidebar.tsx          # Right sidebar
│   │   ├── ContextPanel.tsx          # Context/knowledge panel
│   │   ├── BottomTerminalDock.tsx    # Terminal dock
│   │   ├── SidebarFilesTree.tsx      # File tree view
│   │   ├── RightSidebarTabs.tsx      # Tab management for right sidebar
│   │   └── ... (misc utilities)
│   │
│   ├── chat/                         # Chat interface (34 files)
│   │   ├── ChatContainer.tsx         # Main chat wrapper
│   │   ├── MessageList.tsx           # Message virtualized list
│   │   ├── ChatMessage.tsx           # Individual message renderer
│   │   ├── ChatInput.tsx             # Input with autocomplete
│   │   ├── ModelControls.tsx         # Model/provider selection
│   │   ├── MarkdownRenderer.tsx      # Markdown to React
│   │   ├── DiffPreview.tsx           # Inline diff preview
│   │   ├── FileAttachment.tsx        # File attachment display
│   │   ├── PermissionRequest.tsx     # Permission UI cards
│   │   ├── QuestionCard.tsx          # Question/approval cards
│   │   ├── AgentMentionAutocomplete/ # Agent @mention suggestions
│   │   ├── FileMentionAutocomplete/  # File mention suggestions
│   │   ├── CommandAutocomplete/      # Command suggestions
│   │   ├── message/                  # Message part renderers
│   │   │   └── parts/                # Individual part types
│   │   ├── components/               # Chat-specific components
│   │   ├── hooks/                    # Chat-specific hooks
│   │   ├── lib/                      # Chat utilities
│   │   │   ├── scroll/               # Scroll behavior
│   │   │   └── turns/                # Message turn management
│   │   └── ... (mobile variants)
│   │
│   ├── views/                        # Main view modes (10 files)
│   │   ├── ChatView.tsx              # Chat workspace view
│   │   ├── GitView.tsx               # Git operations view
│   │   ├── DiffView.tsx              # Unified diff viewer
│   │   ├── TerminalView.tsx          # Terminal emulator
│   │   ├── FilesView.tsx             # File browser
│   │   ├── SettingsView.tsx          # Settings panel
│   │   ├── PlanView.tsx              # Plan/document view
│   │   ├── MultiRunWindow.tsx        # Multi-run executor
│   │   ├── SettingsWindow.tsx        # Settings dialog
│   │   ├── agent-manager/            # Agent management panel
│   │   │   ├── AgentManagerView.tsx
│   │   │   ├── AgentManagerSidebar.tsx
│   │   │   ├── AgentGroupDetail.tsx
│   │   │   └── AgentManagerEmptyState.tsx
│   │   ├── git/                      # Git view subcomponents (33 files)
│   │   │   ├── BranchSelector.tsx
│   │   │   ├── ChangesSection.tsx
│   │   │   ├── CommitSection.tsx
│   │   │   ├── HistorySection.tsx
│   │   │   ├── PullRequestSection.tsx
│   │   │   └── ... (git operations UI)
│   │   └── PierreDiffViewer.tsx      # Diff viewer wrapper
│   │
│   ├── session/                      # Session management (20 files)
│   │   ├── SessionSidebar.tsx        # Session list/tree
│   │   ├── SessionDialogs.tsx        # Session action dialogs
│   │   ├── DirectoryTree.tsx         # Directory picker
│   │   ├── DirectoryExplorerDialog.tsx
│   │   ├── BranchPickerDialog.tsx
│   │   ├── NewWorktreeDialog.tsx
│   │   ├── GitHubIntegrationDialog.tsx
│   │   ├── GitHubIssuePickerDialog.tsx
│   │   ├── GitHubPrPickerDialog.tsx
│   │   ├── ProjectNotesTodoPanel.tsx # Notes & TODO tracking
│   │   ├── sidebar/                  # Sidebar sub-components
│   │   │   ├── SidebarHeader.tsx
│   │   │   ├── SidebarFooter.tsx
│   │   │   ├── SessionGroupSection.tsx
│   │   │   ├── SessionNodeItem.tsx
│   │   │   ├── SidebarProjectsList.tsx
│   │   │   └── ... (drag-and-drop integration)
│   │   └── hooks/                    # Session-specific hooks
│   │
│   ├── terminal/                     # Terminal UI components
│   │   └── (minimal - mostly via Ghostty Web)
│   │
│   ├── sections/                     # Settings/config sections (14 dirs)
│   │   ├── agents/                   # Agent configuration
│   │   ├── commands/                 # Command management
│   │   ├── git-identities/           # Git author config
│   │   ├── mcp/                      # MCP protocol config
│   │   ├── openchamber/              # App settings
│   │   ├── projects/                 # Project settings
│   │   ├── providers/                # Provider auth config
│   │   ├── remote-instances/         # Remote deployment config
│   │   ├── skills/                   # Skill catalog
│   │   ├── usage/                    # Quota/usage display
│   │   └── shared/                   # Shared section components
│   │
│   ├── voice/                        # Voice features (6 files)
│   │   └── Voice UI components & provider
│   │
│   ├── multirun/                     # Multi-run execution UI
│   │   └── Parallel execution visualization
│   │
│   ├── comments/                     # Inline comments/annotations
│   │
│   ├── desktop/                      # Desktop-specific features
│   │   └── SSH/host configuration UI
│   │
│   ├── icons/                        # Icon components
│   │   └── FileIcon.tsx (with 300+ file type SVG icons)
│   │
│   ├── auth/                         # Authentication UI
│   │   └── SessionAuthGate.tsx
│   │
│   ├── onboarding/                   # Onboarding flow
│   │   └── CLI installation wizard
│   │
│   ├── mcp/                          # MCP protocol UI
│   │
│   └── providers/                    # React Context providers
│       └── ThemeProvider.tsx
│
├── contexts/                         # React Context (11 files)
│   ├── ThemeSystemContext.tsx        # Theme switching (24KB)
│   ├── theme-system-context.ts       # Type definitions
│   ├── useThemeSystem.ts             # Theme hook
│   ├── DiffWorkerProvider.tsx        # Diff computation worker
│   ├── DrawerContext.tsx             # Mobile drawer state
│   ├── FireworksContext.tsx          # Celebration animation state
│   ├── RuntimeAPIProvider.tsx        # Runtime API injection
│   ├── runtimeAPIContext.ts          # Type defs
│   └── runtimeAPIRegistry.ts         # API registration
│
├── hooks/                            # Custom hooks (42 files)
│   ├── useEventStream.ts             # SSE connection (103KB)
│   ├── useServerSessionStatus.ts     # Session polling
│   ├── useSessionStatusBootstrap.ts  # Startup sync
│   ├── useChatScrollManager.ts       # Auto-scroll behavior
│   ├── useBrowserVoice.ts            # Browser voice API (30KB)
│   ├── useServerTTS.ts               # Server text-to-speech
│   ├── useSayTTS.ts                  # Voice playback
│   ├── useKeyboardShortcuts.ts       # Global keyboard handlers
│   ├── useRouter.ts                  # URL-based routing
│   ├── useMenuActions.ts             # App menu actions
│   ├── useGitPolling.tsx             # Git status polling
│   ├── useGitHubPrBackgroundTracking.ts # PR monitoring
│   ├── useQueuedMessageAutoSend.ts   # Message queuing
│   ├── useSessionAutoCleanup.ts      # Session management
│   ├── useMessageTTS.ts              # Message audio
│   ├── useProviderLogo.ts            # Provider branding
│   ├── useFontPreferences.ts         # Font selection
│   ├── useScrollEngine.ts            # Advanced scrolling
│   ├── usePushVisibilityBeacon.ts    # Visibility tracking
│   ├── usePwaInstallPrompt.ts        # PWA installation
│   ├── usePwaManifestSync.ts         # PWA updates
│   ├── useWindowTitle.ts             # Document title sync
│   ├── useEffectiveDirectory.ts      # Directory resolution
│   ├── useChatSearchDirectory.ts     # Search in directory
│   ├── useDrawerSwipe.ts             # Mobile swipe gestures
│   ├── useEdgeSwipe.ts               # Edge detection swipes
│   ├── useLongPress.ts               # Touch hold detection
│   ├── useDebouncedValue.ts          # Debounce hook
│   ├── useIsTextTruncated.ts         # Text overflow detection
│   ├── useSessionActivity.ts         # Activity tracking
│   ├── useAvailableTools.ts          # Tool discovery
│   ├── useModelLists.ts              # Model filtering
│   ├── usePwaDetection.ts            # PWA detection
│   ├── useVoiceContext.ts            # Voice state
│   ├── useFileSystemAccess.ts        # File API access
│   ├── useAssistantStatus.ts         # Assistant state
│   ├── useAssistantTyping.ts         # Typing indicator
│   ├── useFireworks.ts               # Animation trigger
│   └── useRuntimeAPIs.ts             # Runtime API access
│
├── stores/                           # Zustand stores (36+ files)
│   ├── useSessionStore.ts            # Active session management (core)
│   ├── sessionStore.ts               # Session metadata & CRUD
│   ├── messageStore.ts               # Message streaming & cache
│   ├── fileStore.ts                  # File operations
│   ├── contextStore.ts               # Context/knowledge state
│   ├── permissionStore.ts            # Permission requests
│   ├── questionStore.ts              # Question/approval queue
│   ├── useConfigStore.ts             # Provider/model config
│   ├── useUIStore.ts                 # UI state (sidebar, dialogs)
│   ├── useDirectoryStore.ts          # Current working directory
│   ├── useSessionDisplayStore.ts     # Display preferences
│   ├── useSessionFoldersStore.ts     # Folder organization
│   ├── useSessionFolderDndStore.ts   # Drag-and-drop state
│   ├── useProjectsStore.ts           # Project metadata
│   ├── useTerminalStore.ts           # Terminal state
│   ├── useTodoStore.ts               # TODO tracking
│   ├── useGitStore.ts                # Git state & operations
│   ├── useGitHubAuthStore.ts         # GitHub authentication
│   ├── useQuotaStore.ts              # Usage quota tracking
│   ├── useUpdateStore.ts             # App update state
│   ├── useOpenInAppsStore.ts         # External app integration
│   ├── useSkillsCatalogStore.ts      # Skill registry
│   ├── useAgentGroupsStore.ts        # Agent grouping
│   ├── useDesktopSshStore.ts         # SSH host management
│   ├── useMcpConfigStore.ts          # MCP protocol config
│   ├── useMultiRunStore.ts           # Multi-run execution
│   ├── globalSessions.ts             # Session list (legacy)
│   ├── types/sessionTypes.ts         # Type definitions (session)
│   ├── utils/                        # Store utilities (11 files)
│   │   ├── messageProjectors.ts      # Message filtering/projection
│   │   ├── messageUtils.ts           # Message parsing
│   │   ├── streamingUtils.ts         # Stream lifecycle management
│   │   ├── permissionUtils.ts        # Permission helpers
│   │   ├── permissionAutoAccept.ts   # Auto-approval logic
│   │   ├── contextUtils.ts           # Context extraction
│   │   ├── tokenUtils.ts             # Token counting
│   │   ├── messageProjectors.ts      # Message filtering
│   │   ├── safeStorage.ts            # LocalStorage wrapper
│   │   └── streamDebug.ts            # Stream debugging
│   └── types/                        # Store type definitions
│
├── lib/                              # Utilities & business logic (64 files)
│   ├── api/
│   │   └── types.ts                  # Runtime API interface definitions
│   │
│   ├── codemirror/
│   │   ├── languageByExtension.ts    # File → language mapping
│   │   └── flexokiTheme.ts           # CodeMirror theme
│   │
│   ├── theme/
│   │   ├── themes/
│   │   │   ├── index.ts              # Theme registry
│   │   │   ├── presets.ts            # Built-in themes
│   │   │   ├── prColors.ts           # PR highlighting colors
│   │   │   ├── flexoki-light.json
│   │   │   ├── flexoki-dark.json
│   │   │   ├── fields-of-the-shire-light.json
│   │   │   └── fields-of-the-shire-dark.json
│   │   ├── cssGenerator.ts           # CSS variable generation
│   │   ├── syntaxThemeGenerator.ts   # Syntax highlighting themes
│   │   └── vscode/adapter.ts         # VS Code theme compatibility
│   │
│   ├── git/
│   │   ├── branchNameGenerator.ts    # Auto-generate branch names
│   │   ├── integrateWorktreeCommits.ts
│   │   └── (git utilities)
│   │
│   ├── terminal/
│   │   ├── SerializeAddon.ts         # Terminal serialization
│   │   └── (terminal utilities)
│   │
│   ├── voice/
│   │   ├── voiceSession.ts           # Voice state machine
│   │   ├── voiceConfig.ts            # Voice settings
│   │   ├── voiceHooks.ts             # Voice-related hooks
│   │   ├── browserVoiceService.ts    # Web Audio API
│   │   ├── realtimeClientTools.ts    # Voice API tools
│   │   ├── contextFormatters.ts      # Voice message formatting
│   │   ├── summarize.ts              # Voice conversation summary
│   │   └── index.ts                  # Voice module exports
│   │
│   ├── diff/
│   │   └── workerFactory.ts          # Worker pool management
│   │
│   ├── permissions/
│   │   ├── editModeColors.ts         # Permission level colors
│   │   └── editPermissionDefaults.ts # Default permissions
│   │
│   ├── quota/
│   │   ├── index.ts                  # Quota display logic
│   │   ├── utils.ts                  # Quota calculations
│   │   ├── model-families.ts         # Model grouping
│   │   └── providers/                # Provider-specific quota
│   │
│   ├── messages/
│   │   ├── messageText.ts            # Text extraction
│   │   ├── executionMeta.ts          # Execution fork metadata
│   │   ├── inlineComments.ts         # Comment parsing
│   │   ├── agentMentions.ts          # Agent @mention parsing
│   │   ├── providerAuthError.ts      # Auth error detection
│   │   └── synthetic.ts              # Synthetic message generation
│   │
│   ├── router/
│   │   ├── index.ts                  # Router state machine
│   │   ├── types.ts                  # Route types
│   │   ├── parseRoute.ts             # URL → state parser
│   │   └── serializeRoute.ts         # State → URL serializer
│   │
│   ├── shiki/
│   │   ├── appThemeRegistry.ts       # Theme → Shiki mapping
│   │   ├── textMateThemeFromAppTheme.ts
│   │   └── vscodeTextMateTheme.ts
│   │
│   ├── worktrees/
│   │   ├── worktreeManager.ts        # Worktree lifecycle
│   │   ├── worktreeStatus.ts         # Status polling
│   │   ├── worktreeCreate.ts         # Creation wizard
│   │   ├── worktreeBootstrap.ts      # Initialization
│   │   └── pendingDraftWorktree.ts   # Draft tracking
│   │
│   ├── settings/
│   │   └── metadata.ts               # Settings schema
│   │
│   ├── opencode/
│   │   └── client.ts                 # SDK client wrapper
│   │
│   ├── utils.ts                      # Utility functions (cn, platform detection)
│   ├── clipboard.ts                  # Clipboard operations
│   ├── device.ts                     # Device info detection
│   ├── desktop.ts                    # Desktop/Tauri detection
│   ├── desktopHosts.ts               # Remote host config
│   ├── desktopSsh.ts                 # SSH key management
│   ├── debug.ts                      # Debug utilities
│   ├── gitApi.ts                     # HTTP git API
│   ├── gitApiHttp.ts                 # Git over HTTP
│   ├── terminalApi.ts                # Terminal control API
│   ├── persistence.ts                # LocalStorage wrapper
│   ├── appearancePersistence.ts      # Appearance preferences
│   ├── appearanceAutoSave.ts         # Auto-persist appearance
│   ├── directoryPersistence.ts       # Directory history
│   ├── typographyWatcher.ts          # Font preference watcher
│   ├── modelPrefsAutoSave.ts         # Model preference auto-save
│   ├── projectMeta.ts                # Project metadata
│   ├── projectActions.ts             # Project operations
│   ├── fileTypeIcons.ts              # File type icon mapping
│   ├── fileOpenLimits.ts             # Open file restrictions
│   ├── filesViewShowGitignored.ts    # Gitignore toggle
│   ├── directoryShowHidden.ts        # Hidden file toggle
│   ├── ime.ts                        # IME composition handling
│   ├── pwa.ts                        # PWA utilities
│   ├── openchamberConfig.ts          # App config parsing
│   ├── configSync.ts                 # Config synchronization
│   ├── configUpdate.ts               # Config update handling
│   ├── openCodeStatus.ts             # CLI status detection
│   ├── fontOptions.ts                # Font choice definitions
│   ├── codeTheme.ts                  # Syntax theme selection
│   ├── url.ts                        # URL utilities
│   ├── shortcuts.ts                  # Keyboard shortcut definitions
│   ├── messageCompletion.ts          # Message history search
│   ├── messageCursorPersistence.ts   # Cursor position saving
│   ├── messageFreshness.ts           # Stale message detection
│   ├── agentColors.ts                # Agent color assignment
│   ├── typography.ts                 # Typography utilities
│   ├── userSendAnimation.ts          # Send animation state
│   ├── contextFileOpenGuard.ts       # File open validation
│   ├── fileTypeIconIds.ts            # Icon ID mapping
│   ├── toolHelpers.ts                # Tool execution utils
│   ├── toolStatus.ts                 # Tool status tracking
│   ├── execCommands.ts               # Command execution
│   ├── worktreeSessionCreator.ts     # Worktree session creation
│   └── gitApi.ts                     # Git operations
│
├── assets/
│   └── icons/
│       ├── file-types/               # 300+ SVG file type icons (EXCLUDED from report)
│       └── (other icon assets)
│
├── constants/
│   └── sidebar.ts                    # Sidebar constants
│
└── types/                            # Global TypeScript types (14 files)
    ├── index.ts                      # Main type exports
    ├── theme.ts                      # Theme structure (6240B)
    ├── permission.ts                 # Permission request types
    ├── question.ts                   # Question/approval types
    ├── quota.ts                      # Quota/usage types
    ├── tool.ts                       # Tool execution types
    ├── multirun.ts                   # Multi-run execution types
    ├── worktree.ts                   # Worktree metadata
    ├── ghostty-web.d.ts              # Ghostty terminal types
    ├── vscode.d.ts                   # VS Code extension types
    ├── desktop.d.ts                  # Desktop/Tauri types
    ├── streamdown.d.ts               # Streamdown types
    ├── codemirror-lang-elixir.d.ts  # Elixir language plugin
    └── react-syntax-highlighter-create-element.d.ts
```

---

## 3. Major Components & Features

### Core Feature Modules

#### **Chat/Conversation System**
- `ChatView` - Main conversation interface
- `ChatContainer` - Chat wrapper with scroll management
- `MessageList` - Virtualized message rendering
- `ChatMessage` - Individual message with formatting
- `ChatInput` - Input area with autocomplete & attachments
- `MarkdownRenderer` - Markdown → React rendering
- Message parts: text, code, diff, table, execution results, etc.
- Autocomplete: agents (@), files (#), commands (/)
- File attachments & inline diff preview

#### **Session Management**
- `SessionSidebar` - Session tree/list
- Session CRUD operations (create, rename, delete, restore)
- Folder organization with drag-and-drop
- Project notes & TODO tracking
- Worktree (git branch) support
- Directory picker & explorer

#### **Git Integration**
- `GitView` - Unified git operations panel
- Branch selection & creation
- Changes staging & unstaging
- Commit composition with AI suggestions
- Pull request linking (GitHub)
- Git history & blame
- Conflict resolution
- Worktree management
- PR background tracking & notifications

#### **Diff Viewer**
- `DiffView` - Pierre-based unified diff renderer
- Syntax highlighting for changed code
- Context expansion controls
- Worker-based diff computation

#### **Terminal Emulator**
- `TerminalView` - Ghostty Web-based terminal
- Custom serialization addon
- Integration with file operations
- Workspace isolation per session

#### **File Browser**
- `FilesView` - Directory tree with virtualization
- File type icons (300+ types)
- Gitignore awareness
- Hidden file toggle
- Context menu operations

#### **Settings & Configuration**
- `SettingsView` - In-app settings panel
- Model/provider selection
- Appearance (theme, fonts, typography)
- Keyboard shortcuts
- Agent management
- MCP protocol configuration
- GitHub authentication
- SSH host configuration (desktop)
- Skill catalog browser

#### **Voice Features**
- Browser voice API support
- Server-side TTS streaming
- Voice session state machine
- Real-time voice tools integration
- Conversation summarization

#### **Multi-run Execution**
- Parallel task execution
- Progress visualization
- Result aggregation

#### **Agent & Provider Management**
- Agent catalog display
- Provider authentication dialogs
- Logo/branding integration
- Quota & usage display

#### **VS Code Extension Support**
- Simplified `VSCodeLayout`
- Agent manager panel
- Chat panel in sidebar
- Configuration injection

---

## 4. State Management Architecture

### Zustand Store Patterns

**Core Approach:** Each major feature has a dedicated store using Zustand

```typescript
// Pattern: Create with devtools + persist middleware
export const useXxxStore = create<State>()(
  devtools(
    persist(
      (set, get) => ({
        // State
        value: initial,
        // Actions
        setValue: (v) => set({ value: v }),
      }),
      {
        name: 'xxx-store',
        storage: createJSONStorage(() => safeStorage),
      }
    ),
    { name: 'XxxStore' }
  )
);
```

**Key Stores:**

| Store | Purpose | Size |
|-------|---------|------|
| `useSessionStore` | Active session, current ID, CRUD | Core |
| `messageStore` | Message streaming, caching, deltas | Large (streaming) |
| `fileStore` | File operations, attachments | Medium |
| `contextStore` | Context/knowledge state | Medium |
| `permissionStore` | Permission requests queue | Small |
| `useConfigStore` | Provider/model config, connection | Core |
| `useUIStore` | UI state (sidebar, dialogs, theme) | Medium |
| `useDirectoryStore` | Current working directory | Small |
| `useTerminalStore` | Terminal state per session | Medium |
| `useGitStore` | Git operations, branch state | Medium |
| `useProjectsStore` | Project metadata cache | Medium |
| `useTodoStore` | TODO items & notes | Small |
| `useQuotaStore` | Usage quota & rate limits | Small |
| `useGitHubAuthStore` | GitHub token & auth state | Small |

**Utilities:**
- `messageProjectors.ts` - Filter/revert message history
- `messageUtils.ts` - Extract text, normalize parts
- `streamingUtils.ts` - Manage message streaming lifecycle
- `permissionAutoAccept.ts` - Auto-approval logic
- `safeStorage.ts` - LocalStorage error handling

---

## 5. Theming System

### Theme Architecture

**Multi-layer approach:**

1. **Theme Definition** (`lib/theme/themes/`)
   - 4 built-in themes:
     - Flexoki Light (default)
     - Flexoki Dark (default)
     - Fields of the Shire Light
     - Fields of the Shire Dark
   - Custom user themes via JSON override
   - VSCode theme compatibility layer

2. **CSS Variable Generation** (`CSSVariableGenerator`)
   - Converts theme colors → CSS custom properties
   - Supports light/dark variants
   - Automatic text color contrast adjustment

3. **Theme Context** (`ThemeSystemContext`)
   - Global theme state (mode, light/dark theme IDs)
   - System preference detection
   - Theme switching with transition suppression
   - VS Code theme sync

4. **Syntax Highlighting Themes**
   - Shiki integration for code highlighting
   - TextMate theme generation from app theme
   - CodeMirror theme adaptation
   - VS Code compatibility

5. **Tailwind CSS Integration**
   - CSS variables for design tokens
   - Custom colors in `tailwind.config.js`
   - `tailwind-merge` for class conflicts

**Theme System Features:**
- System preference detection (`prefers-color-scheme`)
- Per-variant theme selection (light vs dark)
- Persistent theme preferences
- Desktop app integration (Tauri)
- VS Code extension theme sync

---

## 6. Styling Approach

### CSS Stack
- **Tailwind CSS 4.0** - Utility-first
- **CSS Custom Properties** - Theme variables
- **Design System CSS** - Global design tokens
- **Mobile-first CSS** - Responsive breakpoints
- **Animated transitions** - Motion library

### Key CSS Files
- `index.css` (1385 lines) - Global styles, animations, utilities
- `design-system.css` (15KB) - Design tokens, color variables
- `typography.css` - Font stacks, text utilities
- `mobile.css` (14KB) - Mobile-specific overrides
- `fireworks.css` - Celebration animation styles

### Design Tokens
- Color palette (primary, surface, interactive, status)
- Typography (sans, mono font stacks)
- Spacing scale (sizes.ts)
- Border radius & shadows
- Z-index scale
- Animation timing

---

## 7. Key Patterns & Conventions

### Component Patterns

**1. Radix UI Wrapper Pattern**
```tsx
// Wrap Radix primitives with custom styling
import * as TooltipPrimitive from "@radix-ui/react-tooltip";
export const Tooltip = React.forwardRef<...>((props, ref) => (
  <TooltipPrimitive.Root>
    <TooltipPrimitive.Trigger>{trigger}</TooltipPrimitive.Trigger>
    <TooltipPrimitive.Content className={cn(...)}>
      {content}
    </TooltipPrimitive.Content>
  </TooltipPrimitive.Root>
));
```

**2. CVA (Class Variance Authority) for Variants**
```tsx
const buttonVariants = cva("base-styles", {
  variants: {
    size: { sm: "...", lg: "..." },
    variant: { primary: "...", ghost: "..." },
  },
});
```

**3. Compound Components**
```tsx
<Dialog>
  <DialogTrigger>Open</DialogTrigger>
  <DialogContent>
    <DialogHeader>Title</DialogHeader>
  </DialogContent>
</Dialog>
```

**4. Hook Composition**
Multiple hooks work together (e.g., `useEventStream` + `useSessionStore` + `useRouter`)

**5. Context + Hook Pattern**
```tsx
const ThemeProvider = ({ children }) => (
  <ThemeContext.Provider value={...}>
    {children}
  </ThemeContext.Provider>
);
const useTheme = () => useContext(ThemeContext);
```

### Store Patterns

**1. Selector-based Usage**
```tsx
const value = useStore((state) => state.value);
```

**2. Middleware Composition**
```tsx
create<State>()(
  devtools(
    persist(
      (set, get) => ({ ... }),
      { name: 'store-key' }
    ),
    { name: 'StoreName' }
  )
);
```

**3. Async Actions**
```tsx
setValue: async (v) => {
  const result = await api.save(v);
  set({ value: result });
}
```

### Styling Conventions

**1. `cn()` Utility for Class Merging**
```tsx
className={cn("base-class", condition && "conditional-class")}
```

**2. Platform Detection**
```tsx
import { isMacOS, isTauriShell, isDesktopShell } from '@/lib/desktop';
```

**3. Font Stack Management**
CSS variables for font families dynamically set in App.tsx

**4. Theme Variables**
```css
var(--color-primary-base)
var(--color-surface-background)
var(--color-interactive-border)
```

### API Integration

**1. Runtime API Injection**
```tsx
// Global window object with runtime APIs
window.__OPENCHAMBER_RUNTIME_APIS__ = { git, github, ... }
```

**2. Registry Pattern**
```tsx
registerRuntimeAPIs(apis);
const apis = getRegisteredRuntimeAPIs();
```

**3. Async Initialization**
App bootstrap awaits: `syncDesktopSettings()`, `initializeAppearancePreferences()`

### Message & Streaming

**1. Part-based Architecture**
Messages decomposed into typed parts (text, code, table, etc.)

**2. Streaming Queues**
Non-blocking queues for streaming updates with flush scheduling

**3. Delta Updates**
Incremental updates to message parts (`field`, `delta`)

**4. Lifecycle Management**
Touch → Active → Completion timeline per message

---

## 8. Layout System

### Three-Pane Desktop Layout
```
┌─────────────────────────────────────────────────┐
│ Header (toolbar, model/agent selector)          │
├──────────┬──────────────────────┬───────────────┤
│          │                      │               │
│ Sidebar  │ Main View (Chat)     │ Right Sidebar │
│          │ (700-920px)          │ (Context,     │
│ Sessions │                      │  Files, Git)  │
│          │                      │               │
│ Projects │                      │               │
│          │ ┌────────────────────┤               │
│          │ │ Terminal Dock      │               │
│          │ │ (Optional)         │               │
├──────────┴──────────────────────┴───────────────┤
└─────────────────────────────────────────────────┘
```

**Responsive Behavior:**
- Desktop: All three panes visible
- Tablet: Collapsible sidebars
- Mobile: Drawer-based navigation
- VS Code: Simplified two-pane layout

**Dynamic Sizing:**
- Sidebar: 250-500px (resizable)
- Right sidebar: 400-860px (resizable)
- Terminal: Auto-hide/show (resizable)
- Motion-based drag handles

---

## 9. Desktop & Platform Support

### Desktop Runtime (Tauri)
- SSH key management for remote hosts
- Desktop settings persistence
- Local file system access
- Desktop-specific keyboard shortcuts (Cmd vs Ctrl)
- Notification system

### VS Code Extension
- Simplified layout without git/terminal
- Agent manager panel (separate)
- Chat panel in sidebar
- Command palette integration
- Configuration through extension settings

### Web/PWA Support
- Progressive Web App installation
- Service worker for offline
- Notification API
- Install prompt handling
- LocalStorage persistence

---

## 10. Performance Optimizations

### Virtualization
- `MessageList` - Virtual scrolling for messages
- `SidebarFilesTree` - Virtual scrolling for file tree
- `react-virtual` from TanStack

### Lazy Loading
- Code splitting per view (ChatView, GitView, etc.)
- Lazy component imports

### Streaming & Batching
- Message part deltas queued and flushed
- Non-text updates batched on RAF
- Cooldown timers for update coalescing

### Memoization
- React.memo for expensive components
- useMemo for derived state
- useCallback for stable function references

### Worker Threads
- Diff computation in Web Worker
- Non-blocking CPU-intensive operations

---

## Unresolved Questions

1. **Shiki Integration**: How is Shiki (syntax highlighter) integrated alongside CodeMirror and Prism? Appears to be used for server-side syntax highlighting in diffs.

2. **Custom Theme Extension**: How do users create/install custom themes beyond built-in presets? JSON override mechanism exists but not documented in code exploration.

3. **Message Part Types**: What are all the message part types? Explored `message/parts/` but didn't enumerate all types.

4. **Worker Pool Management**: How many diff workers are pooled and how are they scaled?

5. **SSE Fallback**: What is the fallback behavior when SSE (`useEventStream`) fails or is unavailable?

6. **Git Worktree Isolation**: How are multiple worktrees managed simultaneously in the same session?

