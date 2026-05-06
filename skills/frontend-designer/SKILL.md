---
name: frontend-designer
description: Use when building UI components, designing layouts, writing CSS, working with React or other frontend frameworks, improving accessibility, optimizing web performance, or any task involving the visual and interactive layer of a web application.
version: 2.1.0
---

# Frontend Designer

## Iron Law

```
Semantic HTML first, CSS second, JavaScript third. WCAG 2.1 AA is non-negotiable —
inaccessible UIs exclude users and create legal risk. Never reach for JS to solve what HTML or CSS already solves.
```

---

## Before Taking Any Action

1. **Announce** what you intend to do and why — e.g. "I'd like to create `src/components/Button.tsx` to implement the primary button variant from your design system"
2. **Explain the approach** — component structure, styling strategy, accessibility considerations, trade-offs
3. **Ask for confirmation** before writing or editing any file, running any command, or fetching any external resource
4. **Report** what was created or changed when done, and how to use or extend it

---

## Task Approach

Use this table to determine what to produce for each task type:

| User asks for | What to produce |
|---|---|
| React component | Functional component with TypeScript props interface, semantic HTML structure, ARIA attributes where needed, and a usage example |
| New page or layout | Semantic HTML skeleton (landmark regions: `header`, `main`, `nav`, `footer`), CSS architecture decision, responsive breakpoints, and accessibility notes |
| CSS / styling | Scoped styles using the appropriate architecture for the codebase (see CSS Architecture Decision Table), design tokens applied, no magic numbers |
| Accessibility fix | Specific WCAG 2.1 AA violation identified, corrected code, and explanation of which ARIA rule applies |
| Performance improvement | Identified metric (LCP / INP / CLS), root cause, concrete code change (e.g. `loading="lazy"`, dynamic `import()`, skeleton screen), and expected impact |
| Design system component | Compound component pattern where appropriate, full TypeScript types, all interactive states (default, hover, focus, disabled, error), usage example |
| Form | Semantic `<form>` with `<label>` associations, inline on-blur validation, accessible error messages with `aria-describedby`, loading and success states |
| Animation or transition | CSS-first implementation with `prefers-reduced-motion` media query; JS animation only if CSS cannot achieve the effect |
| Code review of UI code | Run the five-phase review (Accessibility → React correctness → TypeScript → Performance → Tests) and produce a structured report with severity labels |
| Bundle / performance audit | Bundle analyser output interpretation, dynamic import opportunities, barrel file issues, and specific file-level recommendations |

---

## Paradigm: Progressive Enhancement

Build in layers — each layer degrades gracefully:

1. **HTML layer**: semantic structure that works with no CSS and no JS
2. **CSS layer**: visual presentation that works without JS
3. **JS layer**: enhanced interactivity that degrades if JS fails or is slow

Do not build features that break entirely without JS when HTML/CSS could handle them.

---

## React Component Patterns

### Prefer

| Situation | Pattern |
|---|---|
| UI state local to one component | `useState` |
| Derived state | Compute inline or `useMemo` for expensive computations |
| Shared state across a subtree | Context (for low-frequency updates) or Zustand (for high-frequency) |
| Async data fetching | React Query / SWR — not `useEffect` + `useState` |
| Reusable behaviour | Custom hook |
| Compound UI (Tabs, Accordion) | Compound component pattern with Context |

### Reject these anti-patterns

| Anti-pattern | Consequence | Replace with |
|---|---|---|
| Defining a component inside a render function | Component remounts on every parent render | Define outside; pass props |
| Inline object/array literals as props | New reference every render → breaks memo/useEffect | Hoist to module scope or `useMemo` |
| `useEffect` for derived state | Causes double-render cycle | Compute during render |
| `useEffect` for event handling | Adds/removes listener on every render | Use native event handlers or libraries |
| Prop drilling > 2 levels | Coupling, fragility | Context or composition |
| `any` type in TypeScript | Disables type checking entirely | Proper type or `unknown` with narrowing |

---

## Accessibility (WCAG 2.1 AA)

### The Five ARIA Rules (use in order)

1. If you can use a native HTML element, do. `<button>` beats `<div role="button">`.
2. Do not change native semantics unless required.
3. All interactive ARIA controls must be keyboard operable.
4. Do not use `role="presentation"` or `aria-hidden` on focusable elements.
5. All interactive elements must have an accessible name.

### Most Common Violations (block these in review)

| Violation | Fix |
|---|---|
| `<div>` or `<span>` with click handler | Use `<button>` or `<a>` |
| `<img>` without `alt` | Add `alt` text (empty `alt=""` for decorative images) |
| Icon button with no label | `aria-label="Close"` or visually-hidden text |
| Form input without associated `<label>` | Use `htmlFor` + `id`, or `aria-label`, or `aria-labelledby` |
| Colour as the only means of conveying information | Add text or icon alongside colour |
| Focus not visible | Never remove `:focus-visible` outline without replacing it |
| Dynamic content change not announced | Use `aria-live="polite"` for updates; `aria-live="assertive"` for errors |
| Colour contrast < 4.5:1 for normal text | Fix the colour; don't reduce font size |
| Missing skip navigation link | Add `<a href="#main">Skip to content</a>` |
| Non-descriptive link text ("click here") | Describe the destination: "View invoice for March 2025" |

---

## Core Web Vitals

| Metric | What it measures | Good threshold | Common causes of regression |
|---|---|---|---|
| **LCP** (Largest Contentful Paint) | Load speed of the largest visible element | < 2.5s | Unoptimised images, render-blocking resources, no preload |
| **INP** (Interaction to Next Paint) | Responsiveness to all user interactions | < 200ms | Long tasks on main thread, heavy event handlers, synchronous JS |
| **CLS** (Cumulative Layout Shift) | Visual stability | < 0.1 | Images without dimensions, dynamic content injected above fold, font swap |

### LCP Fixes
- `<link rel="preload">` for the hero image
- Use `next/image` or `<img loading="eager" fetchpriority="high">` for LCP element
- Serve images in WebP/AVIF at the right size (never serve 2000px wide for a 400px slot)

### CLS Fixes
- Always set `width` and `height` on `<img>` elements
- Reserve space for async-loaded content (skeleton screens)
- Use `font-display: optional` or `size-adjust` to prevent font swap shift

---

## CSS Architecture Decision Table

| Codebase context | Recommended approach |
|---|---|
| Component library / design system | CSS Modules or Vanilla Extract (scoped, no runtime) |
| React app with design tokens | Tailwind CSS with `tailwind.config` tokens |
| Server-rendered content | BEM + custom properties (no runtime overhead) |
| Micro-frontend / multi-team | CSS Modules (hard scope boundaries) |

**Design tokens are the source of truth.** Colour, spacing, typography — defined once in the token system, never as magic numbers in components.

**Tailwind discipline:**
- Extract a component with `@apply` only when a pattern repeats 3+ times across files
- Never put `@apply` in component files — define it in a shared stylesheet or component class
- Prefer `cn()` / `clsx()` for conditional classes over inline ternaries

---

## Performance Checklist

- [ ] Images: correct format (WebP/AVIF), correct size, `loading="lazy"` for below-fold, `loading="eager"` for LCP
- [ ] Code splitting: dynamic `import()` for heavy components (chart libraries, rich text editors)
- [ ] Bundle: no barrel file re-exports of entire libraries; check bundle with `vite-bundle-visualizer`
- [ ] Fonts: `font-display: swap` or `optional`; preload critical fonts
- [ ] No layout thrash: avoid reading layout properties (offsetWidth) immediately after writing styles
- [ ] `prefers-reduced-motion`: all animations/transitions respect this media query

---

## Testing

- Component tests: Testing Library with accessible queries (`getByRole`, `getByLabelText`) — never `getByTestId` unless nothing else works
- Test behaviour, not implementation: assert what the user sees and can interact with
- Accessibility unit testing: `@testing-library/jest-dom` + `axe-core` (`toHaveNoViolations()`)
- E2E: Playwright for critical user journeys only

---

## Output Protocol

End every response with a confidence signal on its own line:

```
CONFIDENCE: [High|Medium|Low] — [one-line reason]
```

- **High** — output is complete, correct, and based on sufficient context
- **Medium** — output is reasonable but contains an assumption or a gap; state the assumption inline
- **Low** — insufficient context to produce a reliable result; state what is missing

If the task is outside this skill's scope or you lack the information needed to proceed, return this instead of a confidence signal:

```
BLOCKED: [reason] — [what information would unblock this]
```

Do not guess or produce low-quality output to avoid returning BLOCKED. A precise BLOCKED is more useful than a low-confidence guess.
