---
description: UI/UX design and frontend interaction style guide. Covers design tokens, typography, copy architecture, gesture contracts, interactive states, accessibility, and styling conventions.
---

This is the durable standard for UI/UX engineering across all user-facing interfaces. **Agents doing write operations on UI components, layouts, styling, or interactions must read this file.**

---

## 1. Design Tokens & Visual Scales

Never use arbitrary, magic CSS numbers in components. All visual properties derive from structured tokens.

### A. Spacing Scale (4px / 8px Baseline)
* All layout margins, paddings, and grid gaps must align with an 8-point grid (with 4px for micro-spacing):
  * `0.25rem` (4px) — Micro gaps, badge padding, icon-text gap.
  * `0.5rem` (8px) — Tight padding, chip spacing.
  * `0.75rem` (12px) — Input inner padding, compact cards.
  * `1rem` (16px) — Standard component padding, list item gaps.
  * `1.5rem` (24px) — Container padding, section spacing.
  * `2rem` (32px) — Major section dividers.
  * `3rem+` (48px+) — Page hero spacing, layout section margins.

### B. Typography & Font Families
* **Font Family Stacks:** Always specify robust system fallbacks.
  * **Sans (UI / Body / Headings):** Inter / Geist / System Sans (`system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`).
  * **Mono (Code / Data / Numbers / Keys):** JetBrains Mono / Geist Mono / SF Mono (`ui-monospace, 'Cascadia Code', 'Source Code Pro', Menlo, monospace`).
* **Type Scale & Hierarchy:**
  * `caption / legal`: `0.75rem` (12px), line-height `1rem` (16px).
  * `body-sm`: `0.875rem` (14px), line-height `1.25rem` (20px).
  * `body-base`: `1rem` (16px), line-height `1.5rem` (24px).
  * `heading-sm` (h4/h3): `1.125rem` - `1.25rem` (18-20px), line-height `1.75rem`.
  * `heading-md` (h2): `1.5rem` - `1.875rem` (24-30px), line-height `2rem`.
  * `heading-lg` (h1/Display): `2.25rem+` (36px+), line-height `1.15`, tighter tracking (`-0.02em`).
* **Tabular Numbers:** Always apply `font-variant-numeric: tabular-nums` (or `tabular-nums` class) to timers, metrics, data grids, and counters to prevent layout jitter.

### C. Color & Elevation Tokens
* **Semantic over Primitive:** Reference semantic roles (`color-bg-surface`, `color-text-primary`, `color-border-subtle`, `color-accent-hover`), never raw HEX/RGB inside component styles.
* **Elevation & Layering:** Standardize z-indexes and drop shadows into a strict scale (e.g. `base: 0`, `dropdown: 100`, `sticky: 200`, `overlay: 300`, `modal: 400`, `toast: 500`). Banned arbitrary `z-index: 9999`.

---

## 2. Copy Architecture (`<xxx>Copy.ts`)

UI copy is data, not markup. Hardcoded copy directly inside component JSX/HTML creates maintenance debt, impairs localization, and causes copy-paste inconsistencies.

* **Dedicated Copy Files:** Every feature or domain page maintains a companion copy dictionary:
  * Pattern: `src/data/copy/<feature>Copy.ts` or `src/features/<feature>/<feature>Copy.ts`.
* **Structured Dictionary Shape:**
  ```typescript
  export const billingCopy = {
    header: {
      title: "Subscription & Invoices",
      subtitle: "Manage your tier, payment methods, and billing history.",
    },
    emptyState: {
      title: "No Invoices Found",
      description: "You have not incurred any charges for this billing cycle.",
      action: "Upgrade Plan",
    },
    tooltips: {
      seatLimit: "Contact sales to add more than 50 seats.",
    },
    errors: {
      paymentFailed: "Card authorization failed. Please verify your details.",
    },
    aria: {
      invoiceTable: "Past subscription invoices and receipts",
    },
  } as const;
  ```
* **Rules:**
  * Components import and render `billingCopy.header.title`.
  * Zero inline string literals for labels, buttons, error messages, or tooltips.
  * All static copy objects must use `as const` to guarantee strict type inference and immutability.

### Layman Language & Anti-Jargon Rules
AI agents frequently draft copy using internal developer jargon or stiff machine terminology. User-facing copy must be written in plain, human-first layman language:

* **Ban Engineering & Architecture Jargon in UI:**
  * ❌ *"Invalid payload schema"* → ✅ *"Please check the information you entered."*
  * ❌ *"Entity instantiation failed"* → ✅ *"We couldn't create your project. Please try again."*
  * ❌ *"Query execution timeout exceeded"* → ✅ *"This is taking longer than usual. Please check your connection and retry."*
  * ❌ *"Authentication token expired; re-authenticate"* → ✅ *"Your session ended. Please sign in again."*
  * ❌ *"Resource mutation completed successfully"* → ✅ *"Changes saved."*
* **Active Voice & Outcome-Driven:** State what happened or what action the user needs to take in 1–2 brief sentences. Avoid passive, robotic, or overly verbose explanations.
* **Positive, Blameless Tone:** Never blame the user. Clearly indicate how to recover or resolve an issue rather than just stating that an error occurred.

---

## 3. Gesture Contracts & Touch Interactions

Touch and gesture interfaces require strict state machines and threshold guarantees to avoid ambiguous interactions.

### A. Gesture State Lifecycle
Every continuous gesture (drag, swipe, pull-to-refresh, pinch) must conform to an explicit state machine:
`Idle → Detecting → Tracking → (Committed | Cancelled) → Settle`

* **Detecting (Slop Threshold):** Do not trigger gesture action until movement exceeds a 6–8px slop threshold. This distinguishes intent from a tap.
* **Tracking:** Direct 1:1 visual transform tracking the pointer position with zero input lag.
* **Velocity vs. Displacement Thresholds (Fling vs Drag):**
  * If displacement exceeds a geometric boundary (e.g. > 40% of sheet height) → **Commit**.
  * If displacement is low but gesture flick velocity exceeds threshold (e.g. > 500px/s) → **Commit**.
  * Otherwise → **Cancel** and spring back to rest.

### B. Pointer & Viewport Discipline
* **Hover vs. Touch Separation:** Never hide essential UI controls or information behind hover-only triggers. Gate hover effects behind `@media (hover: hover) and (pointer: fine)`.
* **Touch Target Size:** Minimum hit target is **44 × 44px** (or 48 × 48px), even if the visual icon inside is 16px. Use padding or invisible pseudo-elements to expand the active hitbox.
* **Scroll Interference:** Set `touch-action: pan-y` (or `pan-x` / `none`) explicitly on custom gesture containers so the native browser scrolling engine doesn't hijack custom drags.
* **Rubber-Banding & Bounds:** When pulling beyond natural bounds, apply resistance (logarithmic/decay curve) rather than hard stopping.

---

## 4. State Design & Interactive Feedback

Every dynamic surface must handle the **5 Core UI States**:

| State | Rule |
|---|---|
| **1. Initial / Idle** | Clean base layout before user or network activity. |
| **2. Loading / Skeleton** | Skeletons must mirror actual layout dimensions (no generic spinning wheels in main content areas). Prevents Cumulative Layout Shift (CLS). |
| **3. Empty State** | Must contain: clear icon/illustration + explanatory title + descriptive text + 1 primary recovery CTA. |
| **4. Success / Optimistic** | Immediate visual feedback (< 50ms). Optimistic updates must be paired with automated rollback + error toast on network failure. |
| **5. Error / Degraded** | Human-readable explanation (sanitized, no raw stack traces) + 1 retry button or offline recovery guidance. |

* **Button Feedback:** Disable button and display spinner on submit to prevent duplicate submissions.
* **Perceived Performance:** Avoid full-page blank flash. Show stale data while revalidating (SWR pattern) instead of blanking out the view.

---

## 5. Accessibility Baseline (A11y & WCAG AA)

Accessibility is an engineering invariant, not a post-launch cleanup task.

* **Keyboard Navigation:**
  * All interactive elements must be focusable via `Tab` / `Shift+Tab`.
  * Modals, drawers, and popovers must **trap focus** while open and restore focus to the trigger on close.
  * `Escape` must consistently close overlays, drawers, tooltips, and modal dialogs.
* **Visible Focus:**
  * Banned `outline: none` or `outline: 0` unless replaced with an equally high-contrast `:focus-visible` ring.
* **Contrast Ratios (WCAG AA):**
  * Normal text: minimum **4.5:1** contrast ratio against background.
  * Large text (18pt+ or 14pt bold+) and UI components/borders: minimum **3.0:1** contrast ratio.
* **Semantic Elements & ARIA:**
  * Use native HTML elements (`<button>`, `<a>`, `<nav>`, `<dialog>`, `<main>`) before generic `<div>` with `onClick`.
  * If a custom widget is unavoidable, supply explicit `role`, `aria-expanded`, `aria-haspopup`, `aria-controls`, and keyboard listeners (`Enter` / `Space`).
  * Live status updates (e.g. streaming LLM tokens, alert banners) require `aria-live="polite"` or `aria-live="assertive"`.
* **Reduced Motion:**
  * Respect system accessibility preferences:
    ```css
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after {
        animation-duration: 0.01ms !important;
        animation-iteration-count: 1 !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
      }
    }
    ```

---

## 6. Styling & Class Conventions

Whether using Tailwind, CSS Modules, or Vanilla CSS:

* **Utility Class Ordering:**
  1. Layout & Flow (`display`, `position`, `top/right/bottom/left`, `z-index`, `flex/grid`)
  2. Box Model & Dimensions (`width`, `height`, `margin`, `padding`)
  3. Typography (`font`, `text-size`, `font-weight`, `line-height`, `tracking`, `color`)
  4. Visuals & Decor (`background`, `border`, `rounded`, `shadow`, `opacity`)
  5. Interactive & Transitions (`cursor`, `transition`, `transform`, `hover:`, `focus:`, `active:`)
  6. Responsive variants (`sm:`, `md:`, `lg:`)
* **DRY Styling Primitives:** Never duplicate a 15-class utility string across 10 files. Extract reusable visual patterns into shared primitive components (e.g., `<Button variant="primary">`, `<Badge status="warning">`).
