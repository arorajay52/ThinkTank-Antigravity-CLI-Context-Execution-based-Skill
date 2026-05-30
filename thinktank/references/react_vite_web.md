# React Vite Web Frontend Guidelines (Verified Best Practices)

This document compiles verified best practices from industry-standard resources (e.g. Bulletproof React, Robin Wieruch's structural guidelines) for building scalable and performant Vite-based React web applications.

---

## 1. Evolutionary Folder Architecture (Robin Wieruch)

Do not over-engineer folder hierarchies early. Start simple and allow the structure to evolve organically.

*   **Rule of Flat Structure**: Begin with a flat layout. Extract folders only when a component grows too large or requires isolated sub-components.
*   **Keep Nesting Shallow**: Avoid nesting folders more than two levels deep (e.g. `src/features/auth/components/button.tsx` is fine; do not nest deeper like `src/features/auth/components/buttons/primary/small/...`). If nesting goes deeper, decompose the component logic.
*   **Unidirectional Dependency Flow**: Code dependencies must flow in a single direction:
    `Shared Resources (components, hooks, utils) ──> Domain Features ──> Pages / App Router`
    *   *Critical Rule*: Generic or shared components inside `src/components/` must **never** import from feature folders.
*   **Barrel Files Caution**: Barrel files (using `index.ts` to re-export other files) simplify imports but can degrade tree-shaking performance and dev startup times in Vite. Prefer direct imports (e.g., `import { Button } from '@/components/button'`) if barrel files become a bottleneck.

---

## 2. Feature-Based Modular Architecture (Bulletproof React)

Group code by **domain or feature** rather than by file type. This encapsulates related logic and simplifies refactoring.

### Directory Structure
```text
src/
  app/         # Application setup (router config, global providers, root App)
  components/  # Shared, generic UI components (Buttons, Inputs, Modals)
  features/    # Domain-specific feature modules (e.g., auth, users, dashboard)
  hooks/       # Shared, global hooks (e.g., useLocalStorage, useDebounce)
  lib/         # Library configs (Axios instance, React Query client config)
  types/       # Global TypeScript types
  utils/       # Helper utilities (formatters, date math)
```

### Anatomy of a Feature (`src/features/feature-name/`)
Keep all resources dedicated to a single feature self-contained:
*   `api/`: API fetch hooks (e.g., `useUpdateProfile.ts` using React Query).
*   `components/`: Sub-components specific to this feature.
*   `hooks/`: Custom React hooks for this feature's logic.
*   `types/`: TypeScript definitions scoped to the domain.
*   `index.ts`: The feature's Public API. Export **only** the entry-point components or hooks that the rest of the application is allowed to import.

---

## 3. Component & State Management
*   **Named Exports**: Use named exports (`export const Component = ...`) rather than default exports. This ensures consistent naming across imports and refactoring tooling. Use default exports only for lazy-loaded route pages.
*   **Data Fetching**: Do **not** use raw `useEffect` hooks for data fetching. Use **TanStack Query** (React Query) or **SWR** to manage server cache, background refreshes, and state lifecycle.
*   **Zustand for UI State**: Use Zustand or lightweight Context for client-only UI states (sidebars, theme, dialogs).

---

## 4. Verification Check
*   Run `npm run build` to verify production compiler compatibility.
*   Validate console warnings and error logs in the browser.
