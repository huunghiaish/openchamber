# OpenChamber Code Standards & Conventions

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Scope:** All packages

---

## Overview

OpenChamber enforces consistent code quality, type safety, and developer experience across a monorepo with 4 packages. Standards apply to TypeScript, React, JavaScript, Rust, and CSS.

---

## TypeScript Standards

### Strict Mode (Required)

All TypeScript files use **strict mode** in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

**Enforcement:**
- `yarn run type-check` runs pre-commit (no optional chaining assumptions)
- All external library types must have @types or inline declarations
- Any explicit `any` requires comment explaining why

### File Naming

- **Components:** PascalCase (e.g., `ChatInput.tsx`, `MainLayout.tsx`)
- **Utilities:** camelCase (e.g., `formatDate.ts`, `messageParser.ts`)
- **Stores/Hooks:** camelCase (e.g., `useSessionStore.ts`, `useGitStatus.ts`)
- **Types:** PascalCase in separate `*.types.ts` files (e.g., `message.types.ts`)
- **Tests:** Append `.test.ts` or `.spec.ts` suffix

**Pattern:** Use **kebab-case for directories** with descriptive names:
```
components/
├── chat/
├── session-management/
├── git-operations/
└── terminal-emulator/

lib/
├── api-handlers/
├── state-sync/
└── security-validators/
```

### Type Annotations

**Required for:**
- Function parameters & return types
- Component props (interface-based)
- State/context values
- API response/request shapes

**Example:**
```typescript
// ✅ Good
interface ChatMessage {
  id: string;
  content: string;
  timestamp: Date;
  authorId: string;
}

export function ChatInput({ onSend }: { onSend: (msg: string) => Promise<void> }): JSX.Element {
  // ...
}

// ❌ Bad
export function ChatInput({ onSend }) { // no types
  const send: any = async (msg) => { // any, no param type
    // ...
  }
}
```

### Imports & Path Aliases

**Use path aliases from `tsconfig.json`:**
```json
{
  "compilerOptions": {
    "paths": {
      "@openchamber/ui/*": ["../ui/src/*"],
      "@web/*": ["./src/*"],
      "@/*": ["./src/*"]
    }
  }
}
```

**Import Order:**
```typescript
// 1. External libraries
import React, { useState } from 'react';
import { useSomeHook } from 'zustand';

// 2. Internal packages
import { ChatContainer } from '@openchamber/ui/chat';

// 3. Local modules
import { useGitStatus } from './hooks/useGitStatus';
import { formatDate } from './lib/formatDate';

// 4. Side effects (CSS, setup)
import './ChatView.css';
```

---

## React & Component Patterns

### Functional Components (Required)

All components **must be functional components** with hooks. Class components prohibited.

```typescript
// ✅ Good
export function ChatMessage({ message }: { message: ChatMessage }): JSX.Element {
  const [isEditing, setIsEditing] = useState(false);

  return <div>{message.content}</div>;
}

// ❌ Bad (class component)
export class ChatMessage extends React.Component { }
```

### Hooks Usage

**Rules:**
- Hooks only in function bodies, not conditionally
- Custom hooks extracted to `hooks/` directory
- Use Zustand for global state (not context)
- Prefer useCallback for memoized callbacks

**Example:**
```typescript
// ✅ hooks/useSessionMessages.ts
export function useSessionMessages(sessionId: string) {
  const [messages, setMessages] = useState<ChatMessage[]>([]);

  useEffect(() => {
    fetchMessages(sessionId).then(setMessages);
  }, [sessionId]);

  return { messages };
}

// ✅ Component using hook
export function ChatView() {
  const { messages } = useSessionMessages('123');
  return <MessageList messages={messages} />;
}
```

### Props Interface Naming

```typescript
// ✅ ComponentName + "Props"
interface ChatInputProps {
  onSend: (msg: string) => void;
  placeholder?: string;
  disabled?: boolean;
}

export function ChatInput(props: ChatInputProps): JSX.Element {
  // ...
}

// Also acceptable (inline for simple props)
export function ChatInput({
  onSend,
  placeholder = 'Type...'
}: {
  onSend: (msg: string) => void;
  placeholder?: string;
}): JSX.Element {
  // ...
}
```

### CVA (Class Variance Authority) for Styling

Use CVA for component variants instead of conditional classNames:

```typescript
// ✅ Good (CVA)
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva('px-4 py-2 rounded', {
  variants: {
    variant: {
      primary: 'bg-blue-500 text-white',
      secondary: 'bg-gray-200 text-gray-800',
    },
    size: {
      sm: 'text-sm',
      lg: 'text-lg',
    },
  },
  defaultVariants: { variant: 'primary', size: 'sm' },
});

type ButtonVariants = VariantProps<typeof buttonVariants>;

interface ButtonProps extends ButtonVariants, React.ButtonHTMLAttributes<HTMLButtonElement> {}

export function Button({ variant, size, className, ...props }: ButtonProps) {
  return <button className={buttonVariants({ variant, size, className })} {...props} />;
}

// ❌ Bad (conditional classNames)
export function Button({ variant, size, className }) {
  const classes = `px-4 py-2 rounded ${
    variant === 'primary' ? 'bg-blue-500' : 'bg-gray-200'
  } ${size === 'sm' ? 'text-sm' : 'text-lg'}`;
  return <button className={classes} />;
}
```

---

## State Management (Zustand)

### Store Organization

One store per concern. Example structure:

```typescript
// ✅ stores/session-store.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface Session {
  id: string;
  name: string;
  isActive: boolean;
}

interface SessionState {
  sessions: Session[];
  activeSessions: Set<string>;
  addSession: (session: Session) => void;
  removeSession: (id: string) => void;
  setActive: (id: string) => void;
}

export const useSessionStore = create<SessionState>()(
  devtools(
    persist(
      (set) => ({
        sessions: [],
        activeSessions: new Set(),
        addSession: (session) =>
          set(({ sessions }) => ({ sessions: [...sessions, session] })),
        removeSession: (id) =>
          set(({ sessions, activeSessions }) => ({
            sessions: sessions.filter((s) => s.id !== id),
            activeSessions: new Set([...activeSessions].filter((sid) => sid !== id)),
          })),
        setActive: (id) =>
          set(({ activeSessions }) => ({
            activeSessions: new Set(activeSessions).add(id),
          })),
      }),
      { name: 'session-store' }
    )
  )
);
```

**Rules:**
- Use devtools middleware for debugging
- Use persist middleware for localStorage
- Keep stores immutable
- Derive state in components, not stores (no computed stores)

---

## Styling Standards

### Tailwind CSS v4 + Design Tokens

**Design Tokens** defined in `styles/design-system.css`:

```css
/* design-system.css */
:root {
  --color-primary: #EC8B49;
  --color-primary-hover: #DA702C;
  --color-surface-bg: #100F0F;
  --color-text-primary: #CECDC3;
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
  --radius-lg: 1rem;
  --font-mono: 'IBM Plex Mono', monospace;
}
```

**Usage in Components:**

```typescript
// ✅ Good (design tokens)
export function ChatMessage({ message }: ChatMessageProps) {
  return (
    <div className="bg-[var(--color-surface-bg)] text-[var(--color-text-primary)] p-[var(--space-md)] rounded-[var(--radius-md)]">
      {message.content}
    </div>
  );
}

// ✅ Also good (semantic Tailwind with tokens)
export function Button({ children }: { children: React.ReactNode }) {
  return (
    <button className="px-4 py-2 bg-orange-600 hover:bg-orange-700 rounded-md transition-colors">
      {children}
    </button>
  );
}

// ❌ Bad (hardcoded colors, arbitrary values)
<div className="bg-[#EC8B49] text-[#CECDC3]">
```

**No Arbitrary Values in Components:** Use design tokens or semantic Tailwind classes.

### Mobile-First Responsive

```typescript
// ✅ Mobile-first (smallest screen first)
export function ChatContainer() {
  return (
    <div className="flex flex-col md:flex-row lg:grid lg:grid-cols-3 gap-2 md:gap-4">
      {/* Mobile: full-width column */}
      {/* Tablet: flex row */}
      {/* Desktop: 3-col grid */}
    </div>
  );
}
```

**Breakpoints (Tailwind default):**
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px

---

## File Organization

### Components

```
components/
├── chat/
│   ├── ChatContainer.tsx      # Main wrapper
│   ├── ChatMessage.tsx        # Individual message
│   ├── ChatInput.tsx          # Input area
│   ├── hooks/
│   │   ├── useMessageStream.ts
│   │   └── useChatScroll.ts
│   └── lib/
│       └── messageParser.ts
│
├── ui/
│   ├── button.tsx             # Base UI component
│   ├── dialog.tsx
│   └── ...
│
└── session/
    ├── SessionSidebar.tsx
    ├── SessionItem.tsx
    └── dialogs/
        ├── NewSessionDialog.tsx
        └── RenameSessionDialog.tsx
```

### Utilities & Types

```
lib/
├── api-handlers/             # API interaction
│   ├── session-api.ts
│   ├── git-api.ts
│   └── github-api.ts
├── state-sync/               # Cross-component state sync
│   └── message-sync.ts
├── security-validators/      # Input validation
│   └── path-validator.ts
└── types.ts                  # Shared type definitions

hooks/
├── useSessionStore.ts        # Zustand hook
├── useGitStatus.ts
└── useTerminal.ts
```

---

## Naming Conventions

### Variables & Functions

```typescript
// ✅ Descriptive, clear intent
const activeSessionIds = new Set<string>();
function formatTimestampToISO(date: Date): string { }
const isUserAuthenticated = true;
const messageQueues = new Map<string, Message[]>();

// ❌ Ambiguous
const a = [];
const x = 5;
function process(data) { }
const temp = [];
```

### Constants

```typescript
// ✅ UPPER_SNAKE_CASE for module constants
const MAX_MESSAGE_LENGTH = 10000;
const DEFAULT_TIMEOUT_MS = 5000;
const API_BASE_URL = 'http://localhost:3000/api';

// ✅ camelCase for runtime constants
const defaultTheme = 'dark';
const featureFlags = { enableVoice: true };
```

### Boolean Variables

```typescript
// ✅ Prefix with "is", "has", "can", "should"
const isLoading = false;
const hasError = false;
const canEdit = true;
const shouldRetry = false;

// ❌ Ambiguous
const loading = false;     // Could be a function/value
const error = false;       // Confusing
const permission = true;   // Unclear intent
```

---

## Error Handling

### Try-Catch Pattern

```typescript
// ✅ Good error handling
async function fetchSession(id: string): Promise<Session> {
  try {
    const response = await fetch(`/api/sessions/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch session:', error);
    throw new Error('Unable to load session. Please try again.');
  }
}

// ❌ Bad error handling
async function fetchSession(id: string) {
  const response = await fetch(`/api/sessions/${id}`);
  return response.json(); // No error handling
}
```

### Zod for Validation

```typescript
// ✅ Use Zod for runtime validation
import { z } from 'zod';

const SessionSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  createdAt: z.string().datetime(),
});

type Session = z.infer<typeof SessionSchema>;

export function parseSession(data: unknown): Session {
  return SessionSchema.parse(data);
}

// Or safe parse with error handling
export function trySafeParseSession(data: unknown): Session | null {
  const result = SessionSchema.safeParse(data);
  return result.success ? result.data : null;
}
```

---

## Security Standards

### Path Validation

```typescript
// ✅ Validate paths to prevent traversal
import path from 'path';

export function isPathSafe(basePath: string, userPath: string): boolean {
  const resolved = path.resolve(basePath, userPath);
  return resolved.startsWith(path.resolve(basePath));
}

// ❌ Direct concatenation
const filePath = basePath + '/' + userInput; // Vulnerable to traversal
```

### Secret Management

```typescript
// ✅ Never commit secrets
// Store in environment variables or ~/.opencode/auth.json
const apiKey = process.env.OPENCODE_API_KEY;
if (!apiKey) {
  throw new Error('OPENCODE_API_KEY environment variable required');
}

// ❌ Never hardcode
const apiKey = 'sk-1234567890abcdef'; // EXPOSED!
```

### OAuth Token Storage

```typescript
// ✅ tokens stored in ~/.opencode/auth.json (mode 600)
// Web API: /api/github/auth/start, /api/github/auth/complete (device flow)
// No tokens in localStorage/sessionStorage

// ❌ Token in localStorage
localStorage.setItem('github_token', token); // Vulnerable
```

---

## Comments & Documentation

### Function Documentation

```typescript
// ✅ Document public functions
/**
 * Parses a chat message into structured parts.
 *
 * @param raw - Raw message string from OpenCode
 * @returns Structured message with parts array
 * @throws ParseError if message format is invalid
 *
 * @example
 * const msg = parseMessage('[TOOL:exec]{...}Hello');
 * console.log(msg.parts); // [{ type: 'tool', ... }, { type: 'text', content: 'Hello' }]
 */
export function parseMessage(raw: string): ParsedMessage {
  // ...
}

// ✅ Explain complex logic
function scheduleRetry(attempt: number): number {
  // Exponential backoff: 100ms, 200ms, 400ms, 800ms, 1600ms (capped)
  return Math.min(100 * Math.pow(2, attempt), 5000);
}

// ❌ Over-commenting obvious code
const count = 0; // Initialize count to zero
if (count > 0) { // If count is greater than 0
  // ...
}
```

### TODO Comments

```typescript
// ✅ Specific TODOs with context
// TODO: Optimize message parsing (currently O(n²)) - see issue #123
// FIXME: Handle edge case where session.id is null (happens in 0.1% of cases)
// NOTE: This workaround needed for Safari 15 compatibility, remove in 2026

// ❌ Vague
// TODO: fix this
// HACK: blah
```

---

## Testing Standards

### Unit Tests

```typescript
// ✅ Clear test structure
describe('parseMessage', () => {
  it('should parse text message correctly', () => {
    const msg = parseMessage('Hello world');
    expect(msg.parts).toHaveLength(1);
    expect(msg.parts[0]).toEqual({ type: 'text', content: 'Hello world' });
  });

  it('should handle tool output in message', () => {
    const msg = parseMessage('[TOOL:exec]{output}Done');
    expect(msg.parts).toHaveLength(2);
    expect(msg.parts[0].type).toBe('tool');
  });

  it('should throw ParseError on invalid format', () => {
    expect(() => parseMessage('[INVALID')).toThrow(ParseError);
  });
});

// ❌ Poor test structure
it('test1', () => {
  const x = parseMessage('test');
  expect(x).toBeDefined();
});
```

---

## Linting & Formatting

### ESLint Configuration

```javascript
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactPlugin from 'eslint-plugin-react';
import reactHooksPlugin from 'eslint-plugin-react-hooks';

export default [
  eslint.configs.recommended,
  ...tseslint.configs.strict,
  {
    plugins: { react: reactPlugin, 'react-hooks': reactHooksPlugin },
    rules: {
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      'no-console': ['warn', { allow: ['error', 'warn'] }],
    },
  },
];
```

**Pre-commit Hook:**
```bash
npm run lint --fix  # Auto-fix style issues
npm run type-check  # TypeScript check
npm run test        # Run tests (if changes to lib/)
```

---

## Commit Message Standards

**Format:** [Conventional Commits](https://www.conventionalcommits.org/)

```bash
# ✅ Good
git commit -m "feat(chat): add voice input support"
git commit -m "fix(git): handle stale branch names in checkout"
git commit -m "docs: update deployment guide with systemd example"
git commit -m "refactor(state): consolidate session stores"
git commit -m "test: add unit tests for message parser"

# ❌ Bad
git commit -m "updates"
git commit -m "WIP: trying to fix thing"
git commit -m "Fixed some stuff in chat input"
```

**Scope Examples:** chat, git, terminal, session, auth, api, ui, desktop, vscode

---

## Performance Guidelines

### Bundle Size

- **Target:** Keep chunk size under 1.2 MB
- **Monitor:** `npm run build` shows bundle analysis
- **Lazy Load:** Split views by feature (chat, git, settings, etc.)

### React Rendering

```typescript
// ✅ Memoize expensive components
const ChatMessageMemo = React.memo(ChatMessage);

// ✅ Use useCallback for stable references
const handleSend = useCallback(async (msg: string) => {
  await api.sendMessage(msg);
}, []);

// ✅ Virtualize long lists
import { FixedSizeList } from 'react-window';

export function MessageList({ messages }: MessageListProps) {
  return (
    <FixedSizeList height={600} itemCount={messages.length} itemSize={80}>
      {({ index, style }) => (
        <div style={style}>
          <ChatMessage message={messages[index]} />
        </div>
      )}
    </FixedSizeList>
  );
}
```

### Network Performance

- Max upload: 50 MB
- Max request timeout: 4 minutes
- SSE streams preferred for real-time events
- Compress responses (gzip by default in Express)

---

## Accessibility (A11y)

```typescript
// ✅ Semantic HTML + ARIA
export function Dialog({ title, open, onClose, children }: DialogProps) {
  return (
    <div
      role="dialog"
      aria-labelledby="dialog-title"
      aria-modal={open}
      aria-hidden={!open}
    >
      <h2 id="dialog-title">{title}</h2>
      {children}
      <button aria-label="Close dialog" onClick={onClose}>
        ✕
      </button>
    </div>
  );
}

// ✅ Keyboard navigation — use onKeyDown with role="menu"
// ❌ Inaccessible — <div onClick> without keyboard support
```

---

## TypeScript-Specific Patterns

- **Avoid `any`:** Use `unknown` for uncertain types, generics for reusable functions.
- **Discriminated unions:** Use `type` field for complex message/event types — enables exhaustive `switch` narrowing.
- **Strict mode:** All functions must have type annotations. No implicit any.

---

## Summary Checklist

- [ ] TypeScript strict mode enabled
- [ ] All functions have type annotations
- [ ] Functional components with hooks only
- [ ] Zustand for global state
- [ ] Tailwind + design tokens for styling
- [ ] CVA for component variants
- [ ] Kebab-case directories, PascalCase components
- [ ] Path aliases for imports
- [ ] Error handling with try-catch + Zod validation
- [ ] Security: no hardcoded secrets, path validation
- [ ] Tests for lib/ code
- [ ] Comments for complex logic only
- [ ] Conventional commit messages
- [ ] ESLint pre-commit checks
- [ ] Performance: <1.2 MB chunks, virtualized lists

---

**Maintained by:** OpenChamber Team | **License:** MIT
