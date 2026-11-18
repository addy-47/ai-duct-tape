---
description: TypeScript / React (frontend) code style guide and engineering standards. Agents doing write operations on frontend code should read this before modifying code.
---

This is the durable coding standard for the frontend in this project's ecosystem. **Agents doing write operations must read this file alongside `style-guides/design.md` before modifying frontend code.**

---

## 1. Tooling

- **Package Manager:** Always use `pnpm`, never `npm` or `yarn`.
- **Verification:** Run `pnpm lint` and `pnpm build` after every modification. Zero warnings/errors permitted.

## 2. Data & Content Separation

- **Zero Hardcoded Text / Labels:** Banned inline hardcoded strings, labels, select options, or mock objects inside components/pages.
- **Copy Architecture (`<xxx>Copy.ts`):** Domain and feature copy lives in dedicated copy dictionaries (`src/data/copy/<feature>Copy.ts` or `<Feature>Copy.ts`) using `as const`. Components import and consume copy tokens, never inline string literals.
- **Mock & static UI data live in `src/data/`** — never defined inline in components or individual files.

## 3. Layering & Boundaries

- **Strict Service Layer Boundary:** Banned raw API calls (fetch, IPC invoke, etc.) inside React components. All network/IPC calls MUST pass through dedicated service modules in `src/services/`.
- **Components consume data; they do not own or fetch it directly.**
- **Page Responsibility (Layout Only):** Files in `src/pages/` MUST only define visual structure, routing, and layout composition. Heavy business logic, state sync, and data transformations belong in `src/services/`, `src/hooks/`, or `src/store/`.
- **Modular Component Subdirectories:** `src/shared/components/` must be structured into logical feature/domain subdirectories (e.g. `layout/`, `home/`, `settings/`, `common/`). Banned flat, uncategorized component directories.

## 4. State Management

- **Shared state:** `src/context/` or a global store (e.g. Zustand `src/store/`) for low-frequency global state (auth, theme, user preferences, feature flags). Never use context for fast-changing values — it causes full subtree re-renders. Those belong in local state or refs.
- **Reusable stateful logic:** `src/hooks/` when the same logic appears in 2+ components, or when effect/state logic inside a component grows complex enough to obscure what the component renders. Components should read as layout + wiring, not logic.
- **Transient UI state:** local component state/refs.

## 5. Type Safety & Quality

- **Strict TypeScript.** `any` is strictly prohibited — define explicit interfaces/types for all props and service returns. If types get complex, define them explicitly. `any` is treated the same as a hardcoded secret: flag it, fix it.
- **Component Consolidation & Deduplication:** Audit and merge components performing identical or near-identical visual/functional tasks into clean, configurable shared primitives.
- **5 Core UI States:** Every dynamic data view must explicitly handle: Initial/Idle, Loading/Skeleton, Empty State (with CTA), Success/Active, and Error/Offline (with retry action). Never assume a service call succeeds.

## 6. Design System & UX Compliance

- **Refer to `style-guides/design.md`** for design tokens, typography scales, accessibility (WCAG AA), gesture contracts, and utility class conventions.
- **The design system is a closed system:** a fixed set of emphasis/elevation levels, each with a defined purpose. Do not add a new level for a one-off layout problem.
- **Reach for the system's primitives before conventional widgets.** Composition over invention.
- **Expression never outranks usability.** When they conflict, usability wins — and say so.
- **Performance is not optional:** visually heavy components get memoized; animation loops throttle with activity; no unnecessary re-renders.
- **Touch Targets:** Minimum 44x44px interactive hitbox for touch/pointer targets.

---

## 7. React Lifecycle and Performance Invariants

Violating these causes re-render cascades, memory leaks, or unmount crashes.

### 7.1 Hook order

- Every hook runs unconditionally at the top of the component body.
- Never place a hook after an early return. Put the guard in a parent wrapper, or after all hooks.
- Never short-circuit hooks (`useA() ?? useB()`). Call each hook at root level, then coalesce the results.

### 7.2 State and selectors

- Use targeted scalar selectors or shallow comparison. Never destructure a whole store into local variables; that re-renders on unrelated mutations.
- Keep referentially stable actions separate from volatile data. Memoize the action object once; never re-create dispatchers on a streaming tick.
- High-frequency values — stream tokens, waveform levels, live buffers — belong in local state, refs, or dedicated store slices. Never in React Context.
- Scope dirty tracking in settings forms to the individual category. A draft change in one category must not invalidate unrelated tabs.

### 7.3 Lazy mounting and dormant subtrees

- In mapped or virtualized lists, mount expensive auxiliary subtrees — menus, popovers, portals, tooltips — only when triggered (`menuOpen && <...>`). Never mount hundreds of dormant positioning calculators or window listeners.
- Closed drawers, modals, and inactive tabs do zero work: no IPC calls, no timers, no animation frames, no heavy grouping.
- Keep floating UI hooks inactive while closed. Do not run positioning computations for hidden popovers.

### 7.4 Timers, effects, and render purity

- Track every timer and interval ID in a ref and clear it during effect cleanup. Never launch an untracked timer from a cleanup return.
- Never call a timer, interval, or service trigger inside a state updater function.
- Animation frame loops read targets from mutable refs, not from stale closures.
- Never read or mutate a ref during render. Move that work to an effect, layout effect, or event handler.
- Never call `setState` synchronously in an effect body. Adjust state during render from previous state, or defer asynchronously.

### 7.5 Layout reads and compositor cost

- Never read layout-inducing properties (`scrollHeight`, `offsetWidth`, `getBoundingClientRect`) inside a high-frequency render or streaming effect. Guard such reads with a length threshold or debounce them.
- Do not apply `backdrop-filter` through universal or descendant wildcard selectors. Every descendant becomes its own compositor layer.
- Full-viewport SVG filters need explicit tile dimensions, hardware isolation, and containment. Never stack duplicate ambient filter layers.
- Do not apply GPU hints (`transform-gpu`, `will-change`) to hundreds of list items at once.
- For large catalogs, render only the above-the-fold slice on the first frame. Defer the rest through idle callbacks or scroll triggers.

### 7.6 Error contracts and overlays

- Never branch UI logic on `error.message` using regex, `.includes()`, or `match()`. Require a typed failure code from the backend. Where the contract is missing, tag the site with `backend-gap:` and notify the backend owner.
- All floating controls — popovers, dropdowns, inline search — register with the project's overlay manager. Escape closes the innermost control only; it must not dismiss the parent modal.
- Outside-click handlers check the overlay stack depth before closing a parent view.
- Clickable containers use a real `button`, or provide `role="button"`, `tabIndex={0}`, and `onKeyDown`. Never nest interactive elements inside a button.

### 7.7 GPU resource teardown

- Cancel animation frame loops, disconnect observers, and dispose GPU resources on unmount.
- For canvas surfaces, reset dimensions and force context loss before disposing the renderer, then null out dense reference arrays.
- Hoist math and color helper objects into refs. Zero allocations inside an active frame callback.

