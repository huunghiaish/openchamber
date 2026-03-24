# OpenChamber Design Guidelines

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Scope:** UI/UX Standards

---

## Design System Overview

OpenChamber uses a **token-based design system** with Tailwind CSS v4 and CSS custom properties. All visual decisions derive from a unified color palette, typography system, and component library.

**Design Principles:**
1. **Consistency** — Same component, same behavior everywhere
2. **Clarity** — Visual hierarchy guides user attention
3. **Accessibility** — WCAG 2.1 AA as minimum standard
4. **Performance** — Fast renders, smooth interactions
5. **Responsiveness** — Works across desktop, tablet, mobile

---

## Color System (Flexoki)

### Primary Colors

**Flexoki Dark Theme (Default):**
```css
--color-primary: #EC8B49              /* Orange accent */
--color-primary-hover: #DA702C        /* Darker on hover */
--color-primary-active: #F9AE77       /* Lighter on active */

--color-surface-bg: #100F0F            /* Near-black background */
--color-surface-fg: #CECDC3            /* Off-white text */
--color-surface-muted: #1C1B1A90       /* Muted background (with alpha) */
--color-surface-elevated: #1C1A1990    /* Slightly elevated surface */
--color-surface-overlay: #00000080     /* Overlay mask */
```

**Flexoki Light Theme:**
```css
--color-primary: #EC8B49              /* Same orange */
--color-surface-bg: #FFFCF0            /* Off-white background */
--color-surface-fg: #100F0F            /* Dark text */
--color-surface-muted: #D0CEBE99       /* Muted (with alpha) */
```

### Status Colors

```css
/* Success (Green) */
--color-success: #A0AF54
--color-success-fg: #100F0F
--color-success-bg: #66800B20
--color-success-border: #66800B50

/* Error (Red) */
--color-error: #D14D41
--color-error-fg: #100F0F
--color-error-bg: #AF302920
--color-error-border: #AF302950

/* Warning (Orange) */
--color-warning: #DA702C
--color-warning-fg: #100F0F
--color-warning-bg: #BC521520
--color-warning-border: #BC521550

/* Info (Blue) */
--color-info: #4385BE
--color-info-fg: #100F0F
--color-info-bg: #205EA620
--color-info-border: #205EA650
```

### Interactive Colors

```css
--color-interactive-border: #343331
--color-interactive-border-hover: #403E3C
--color-interactive-border-focus: #EC8B49
--color-interactive-selection: #f4f4f41f
--color-interactive-focus: #EC8B49
--color-interactive-focus-ring: #EC8B4950
--color-interactive-cursor: #CECDC3
--color-interactive-hover: #ffffff18
--color-interactive-active: #ffffff1f
```

### PR Status Colors

```css
--color-pr-open: #A0AF54       /* Green */
--color-pr-draft: #878580      /* Gray */
--color-pr-blocked: #DA702C    /* Orange */
--color-pr-merged: #8B7EC8     /* Purple */
--color-pr-closed: #D14D41     /* Red */
```

---

## Typography System

### Font Family

```css
/* Monospace (code, terminal, terminal view) */
--font-mono: 'IBM Plex Mono', 'Monaco', 'Courier New', monospace

/* Sans (UI, labels, headings) */
--font-sans: system-ui, -apple-system, 'Segoe UI', 'Roboto', sans-serif
```

### Font Sizes

```css
/* Scale: 0.875rem → 1rem → 1.125rem → 1.25rem → 1.5rem → 1.875rem → 2.25rem */
--text-xs:   0.75rem    /* 12px - small labels */
--text-sm:   0.875rem   /* 14px - body text, UI */
--text-base: 1rem       /* 16px - standard body */
--text-lg:   1.125rem   /* 18px - subheadings */
--text-xl:   1.25rem    /* 20px - headings */
--text-2xl:  1.5rem     /* 24px - section headers */
--text-3xl:  1.875rem   /* 30px - page titles */
--text-4xl:  2.25rem    /* 36px - hero text */
```

### Line Heights

```css
--leading-tight: 1.2     /* 20% below font size - compact */
--leading-normal: 1.5    /* 50% below font size - readable */
--leading-loose: 1.8     /* 80% below font size - spacious */
```

**Usage:**
- Headings: `--leading-tight`
- Body text: `--leading-normal`
- Quotes/emphasis: `--leading-loose`

### Font Weights

```css
--font-light: 300       /* Not often used */
--font-normal: 400      /* Default body */
--font-medium: 500      /* Subtle emphasis */
--font-semibold: 600    /* Button text, strong emphasis */
--font-bold: 700        /* Headings, critical info */
```

---

## Spacing System

**Base Unit:** `0.25rem` (4px)

```css
--space-0:   0          /* 0px */
--space-px:  1px        /* Borders */
--space-xs:  0.25rem    /* 4px - minimal */
--space-sm:  0.5rem     /* 8px - compact */
--space-md:  1rem       /* 16px - default */
--space-lg:  1.5rem     /* 24px - loose */
--space-xl:  2rem       /* 32px - spacious */
--space-2xl: 3rem       /* 48px - very spacious */
--space-3xl: 4rem       /* 64px - hero spacing */
```

**Usage Guidelines:**
- **Padding inside components:** `--space-md` (16px)
- **Margin between sections:** `--space-lg` to `--space-2xl`
- **Compact lists:** `--space-sm` between items
- **Spacious layout:** `--space-xl` or more

### Gap in Grids/Flex

```css
--gap-xs: 0.5rem        /* 8px - tight layout */
--gap-sm: 1rem          /* 16px - normal */
--gap-md: 1.5rem        /* 24px - loose */
--gap-lg: 2rem          /* 32px - spacious */
```

---

## Border Radius

```css
--radius-none:   0
--radius-xs:     0.125rem   /* 2px - subtle */
--radius-sm:     0.25rem    /* 4px - default buttons */
--radius-md:     0.5rem     /* 8px - cards, inputs */
--radius-lg:     1rem       /* 16px - large modals */
--radius-full:   9999px     /* fully rounded (pills) */
```

**Usage:**
- Buttons: `--radius-sm`
- Input fields: `--radius-sm`
- Cards/panels: `--radius-md`
- Badges/pills: `--radius-full`

---

## Component Patterns

### Button Variants (CVA)

```typescript
// Base styles
const buttonBase = 'px-4 py-2 font-semibold rounded-[var(--radius-sm)] transition-colors';

// Primary button
<button className={`${buttonBase} bg-[var(--color-primary)] text-[var(--color-surface-bg)] hover:bg-[var(--color-primary-hover)]`}>
  Send Message
</button>

// Secondary button
<button className={`${buttonBase} bg-[var(--color-surface-elevated)] text-[var(--color-surface-fg)] hover:bg-[var(--color-interactive-hover)]`}>
  Cancel
</button>

// Danger button
<button className={`${buttonBase} bg-[var(--color-error)] text-[var(--color-error-fg)] hover:bg-red-700`}>
  Delete
</button>

// Ghost button
<button className={`${buttonBase} bg-transparent text-[var(--color-primary)] hover:bg-[var(--color-interactive-hover)]`}>
  Learn More
</button>
```

### Input Fields

```typescript
// Standard input
<input
  className="w-full px-3 py-2 border border-[var(--color-interactive-border)] rounded-[var(--radius-sm)] bg-[var(--color-surface-bg)] text-[var(--color-surface-fg)] focus:outline-none focus:border-[var(--color-interactive-border-focus)] focus:ring-2 focus:ring-[var(--color-interactive-focus-ring)]"
  placeholder="Enter text..."
/>

// Textarea
<textarea
  className="w-full px-3 py-2 border border-[var(--color-interactive-border)] rounded-[var(--radius-sm)] bg-[var(--color-surface-bg)] text-[var(--color-surface-fg)] focus:outline-none focus:border-[var(--color-primary)]"
  rows={4}
/>
```

### Cards/Panels

```typescript
// Standard card
<div className="p-[var(--space-md)] bg-[var(--color-surface-elevated)] rounded-[var(--radius-md)] border border-[var(--color-interactive-border)]">
  <h3 className="text-lg font-semibold text-[var(--color-surface-fg)]">Card Title</h3>
  <p className="text-sm text-[var(--color-surface-muted)] mt-[var(--space-sm)]">
    Card content
  </p>
</div>
```

---

## Layout Patterns

### Three-Pane Layout (Desktop)

**Structure:**
```
┌─────────────────────────────────────────┐
│ Header (toolbar, session switcher)      │
├─────────────────────────────────────────┤
│       │                       │          │
│ Sidebar │ Main (chat/git/etc) │ Right  │
│  (300px) │    (flex-grow)     │ (400px) │
│       │                       │          │
└─────────────────────────────────────────┘
```

**CSS:**
```css
.layout-three-pane {
  display: flex;
  height: 100vh;
  flex-direction: column;
}

.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.sidebar {
  width: 300px;
  overflow-y: auto;
  border-right: 1px solid var(--color-interactive-border);
}

.center {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.right-sidebar {
  width: 400px;
  overflow-y: auto;
  border-left: 1px solid var(--color-interactive-border);
}
```

### Responsive Behavior

**Mobile (< 768px):**
- Stack sidebar, center, right-sidebar vertically
- Hide right sidebar by default (toggle via button)
- Full-width main content

**Tablet (768px - 1024px):**
- Sidebar + center visible
- Right sidebar collapsed (drawer)

**Desktop (> 1024px):**
- All three panes visible
- Draggable dividers for resizing

---

## Typography in UI

### Headings

**Page Title (h1):**
```typescript
<h1 className="text-3xl font-bold text-[var(--color-surface-fg)]">
  Sessions
</h1>
```

**Section Header (h2):**
```typescript
<h2 className="text-xl font-semibold text-[var(--color-surface-fg)] mt-[var(--space-lg)]">
  Git History
</h2>
```

**Subsection (h3):**
```typescript
<h3 className="text-lg font-semibold text-[var(--color-surface-fg)]">
  File Changes
</h3>
```

### Body Text

**Default paragraph:**
```typescript
<p className="text-sm text-[var(--color-surface-fg)] leading-[var(--leading-normal)]">
  Lorem ipsum dolor sit amet...
</p>
```

**Muted/secondary text:**
```typescript
<p className="text-xs text-[var(--color-surface-muted)]">
  Secondary information
</p>
```

**Code/monospace:**
```typescript
<code className="font-mono text-sm bg-[var(--color-surface-elevated)] px-2 py-1 rounded">
  npm install
</code>
```

---

## Responsive Design

### Mobile-First Breakpoints

```css
/* Mobile (default) */
.container {
  display: flex;
  flex-direction: column;
}

/* Tablet (768px+) */
@media (min-width: 768px) {
  .container {
    flex-direction: row;
  }
}

/* Desktop (1024px+) */
@media (min-width: 1024px) {
  .sidebar {
    width: 300px;
  }
}

/* Large Desktop (1280px+) */
@media (min-width: 1280px) {
  .sidebar {
    width: 400px;
  }
}
```

### Touch-Friendly Sizes

- **Minimum touch target:** 44×44px (recommended)
- **Spacing between targets:** 8px minimum
- **Buttons:** 48×48px ideal for thumbs
- **Form inputs:** 44px height minimum

---

## Animations & Transitions

### Standard Transitions

```css
/* Fade in/out */
.fade {
  transition: opacity 200ms ease-in-out;
}

/* Slide (up/down/left/right) */
.slide {
  transition: transform 300ms ease-in-out;
}

/* Color change (hover) */
.color-change {
  transition: background-color 150ms ease-in-out, color 150ms ease-in-out;
}

/* All properties (use sparingly) */
.smooth {
  transition: all 200ms ease-in-out;
}
```

### Timing Functions

- **ease-in-out:** Default (most natural)
- **ease-in:** Starting slow (page transitions)
- **ease-out:** Ending slow (closing modals)
- **linear:** Consistent speed (progress bars)

### Accessibility: Respect Prefers Reduced Motion

```typescript
// Disable animations for accessibility
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

const animationDuration = prefersReducedMotion ? 0 : 300;

<div
  className="transition-all"
  style={{
    transitionDuration: `${animationDuration}ms`,
    transform: isOpen ? 'translateY(0)' : 'translateY(-10px)',
  }}
>
  {children}
</div>
```

---

## Accessibility (A11y)

### Semantic HTML

```typescript
// ✅ Good
<header>Header</header>
<nav>Navigation</nav>
<main>Main content</main>
<aside>Sidebar</aside>
<article>Content</article>
<footer>Footer</footer>

// ❌ Bad
<div className="header">Header</div>
<div className="nav">Navigation</div>
```

### ARIA Attributes

```typescript
// Dialog modal
<div
  role="dialog"
  aria-labelledby="dialog-title"
  aria-modal={isOpen}
  aria-hidden={!isOpen}
>
  <h2 id="dialog-title">Confirm Action</h2>
  {/* content */}
</div>

// Form labels
<label htmlFor="email">Email Address</label>
<input id="email" type="email" />

// Icon-only buttons
<button aria-label="Close dialog">
  <XIcon />
</button>

// Live regions (announcements)
<div role="status" aria-live="polite">
  Changes saved successfully
</div>
```

### Keyboard Navigation

```typescript
// Trap focus in modal
<div
  onKeyDown={(e) => {
    if (e.key === 'Escape') {
      onClose();
    }
    // TAB handling for modal focus trap
  }}
>
  {/* Modal content */}
</div>

// Menu keyboard support
<div
  role="menu"
  onKeyDown={(e) => {
    if (e.key === 'ArrowDown') {
      moveFocusNext();
    } else if (e.key === 'ArrowUp') {
      moveFocusPrevious();
    } else if (e.key === 'Enter') {
      selectCurrent();
    }
  }}
>
  {/* Menu items */}
</div>
```

### Color Contrast

**WCAG 2.1 Requirements:**
- **Normal text:** Minimum 4.5:1 contrast ratio
- **Large text:** Minimum 3:1 contrast ratio
- **UI components:** Minimum 3:1 contrast ratio

**Testing:**
```bash
# Use WebAIM Contrast Checker
# https://webaim.org/resources/contrastchecker/

# Or browser tools
# DevTools → Rendering → Emulate CSS media feature prefers-color-scheme
```

---

## Dark Mode Support

### Automatic Theme Detection

```typescript
// Detect system preference
const isDarkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;

// Listen for changes
window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
  setTheme(e.matches ? 'dark' : 'light');
});
```

### Manual Theme Toggle

```typescript
// Settings → Theme selector
<select onChange={(e) => setTheme(e.target.value)}>
  <option value="system">System</option>
  <option value="light">Light</option>
  <option value="dark">Dark</option>
</select>
```

### CSS Implementation

```css
/* Light theme (default or explicitly set) */
:root,
[data-theme='light'] {
  --color-surface-bg: #FFFCF0;
  --color-surface-fg: #100F0F;
  --color-primary: #EC8B49;
}

/* Dark theme */
@media (prefers-color-scheme: dark),
[data-theme='dark'] {
  :root {
    --color-surface-bg: #100F0F;
    --color-surface-fg: #CECDC3;
    --color-primary: #EC8B49;
  }
}
```

---

## Custom Themes (User-Defined)

**Location:** `~/.config/openchamber/themes/*.json`

**Example theme structure:**
```json
{
  "metadata": {
    "id": "my-theme",
    "name": "My Custom Theme",
    "variant": "dark",
    "version": "1.0.0"
  },
  "colors": {
    "primary": { "base": "#...", "hover": "#...", ... },
    "surface": { "background": "#...", ... },
    "status": { "error": "#...", "success": "#...", ... }
  },
  "typography": {
    "fontFamily": "...",
    "fontSize": { "sm": "...", "base": "..." }
  },
  "spacing": { "xs": "...", "sm": "...", ... }
}
```

**Hot Reload:** Settings → Appearance → Reload Themes (no restart needed)

See [CUSTOM_THEMES.md](./CUSTOM_THEMES.md) for detailed theme guide.

---

## Component Library

### Base UI Components

Located in `packages/ui/src/components/ui/`:

| Component | File | Purpose |
|-----------|------|---------|
| Button | button.tsx | Primary CTA |
| Dialog | dialog.tsx | Modal dialogs |
| Dropdown | dropdown-menu.tsx | Dropdown menus |
| Select | select.tsx | Select dropdowns |
| Tooltip | tooltip.tsx | Hover tooltips |
| Input | textarea.tsx | Text inputs |
| Scroll Area | scroll-area.tsx | Custom scrollbars |
| Command | command.tsx | Command palette base |

### Composition Pattern

**All components use CVA for variants:**
```typescript
import { cva, type VariantProps } from 'class-variance-authority';

const buttonVariants = cva('px-4 py-2 font-semibold rounded transition-colors', {
  variants: {
    variant: {
      primary: 'bg-[var(--color-primary)] text-white',
      secondary: 'bg-gray-200 text-gray-900',
    },
    size: {
      sm: 'text-sm h-8',
      md: 'text-base h-10',
      lg: 'text-lg h-12',
    },
  },
  defaultVariants: { variant: 'primary', size: 'md' },
});

type ButtonVariants = VariantProps<typeof buttonVariants>;

export function Button({ variant, size, className, ...props }: ButtonVariants & React.ButtonHTMLAttributes<HTMLButtonElement>) {
  return (
    <button className={buttonVariants({ variant, size, className })} {...props} />
  );
}
```

---

## Icons

**Icon Library:** Remix Icon 4.7.0

**Usage:**
```typescript
import { ArrowRightIcon, CheckIcon, AlertIcon } from 'remixicon-react';

// Size variants
<ArrowRightIcon size={16} />     {/* 16px - small */}
<ArrowRightIcon size={24} />     {/* 24px - normal */}
<ArrowRightIcon size={32} />     {/* 32px - large */}

// Color
<CheckIcon color="var(--color-success)" />

// In buttons
<button>
  <CheckIcon size={20} className="mr-2" />
  Save
</button>
```

---

## Performance Guidelines

### CSS

- Use CSS variables (no runtime parsing)
- Prefer Tailwind classes (pre-compiled)
- Avoid inline styles (except dynamic values)
- Leverage CSS containment (large lists)

### Animations

- Keep animations < 300ms (feels snappy)
- Use `transform` & `opacity` (GPU-accelerated)
- Avoid animating `width`, `height` (layout thrashing)
- Throttle scroll events with `requestAnimationFrame`

### Images

- Use `.webp` with `.png` fallback
- Responsive images via `srcset`
- Lazy load images below fold
- Optimize SVGs (remove metadata)

---

## Summary Checklist

- [ ] Use design tokens for all colors, spacing, typography
- [ ] Minimum 4.5:1 contrast ratio for text
- [ ] Touch targets 44×44px minimum
- [ ] Mobile-first responsive design
- [ ] Semantic HTML + ARIA attributes
- [ ] Respect `prefers-reduced-motion`
- [ ] Keyboard navigation support
- [ ] Dark mode support
- [ ] CVA for component variants
- [ ] Tailwind for styling (no arbitrary values in components)
- [ ] Consistent spacing & alignment
- [ ] Accessible form labels
- [ ] Loading states for async operations
- [ ] Error states with clear messaging

---

**Maintained by:** OpenChamber Team | **License:** MIT
