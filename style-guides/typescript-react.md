---
description: TypeScript / React (frontend) code style guide and engineering standards. Agents doing write operations on frontend code should read this before modifying code.
---

This is the durable coding standard for the frontend in this project's ecosystem. **Agents doing write operations must read this file before modifying frontend code.**

---

## 1. Tooling

- **Package Manager:** Always use `pnpm`, never `npm` or `yarn`.
- **Verification:** Run `pnpm lint` and `pnpm build` after every modification. Zero warnings/errors permitted.

## 2. Data & Content

- **Zero Hardcoded Text / Labels:** Banned inline hardcoded strings, labels, select options, or mock objects inside components/pages. All static content must live in `src/data/` (e.g. `appData.ts`, `settingsDomains.ts`).
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
- **Never assume a service call succeeds:** loading state, error state, and backend-not-ready state are the default cases to design for, not optional edge cases.

## 6. Design System Compliance

- **The design system is a closed system:** a fixed set of emphasis/elevation levels, each with a defined purpose. Do not add a new level for a one-off layout problem.
- **Reach for the system's primitives before conventional widgets.** Composition over invention.
- **Expression never outranks usability.** When they conflict, usability wins — and say so.
- **Performance is not optional:** visually heavy components get memoized; animation loops throttle with activity; no unnecessary re-renders.
