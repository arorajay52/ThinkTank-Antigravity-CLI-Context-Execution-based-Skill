# React Vite (Web Frontend) Technical Guidelines

This document outlines verified, high-leverage architectural patterns and guidelines for building web applications using React Vite. Consult this file **only** when working on web-targeted features.

---

## 1. Feature-Based Architecture (Bulletproof React)

Organize code by **features** rather than by file type. This encapsulates related logic and makes the codebase highly scalable.

### Directory Structure
```text
src/
  app/         # Application-level setup (router, global providers, main App component)
  assets/      # Static assets (images, icons)
  components/  # Shared, reusable UI components used across features (Buttons, Inputs, Layouts)
  config/      # Global configurations and environment variables
  features/    # Feature-specific modules (self-contained logic)
  hooks/       # Shared, global hooks
  lib/         # Pre-configured instances of third-party libraries (e.g., Axios, TanStack Query client)
  providers/   # Global context providers wrapping the app
  types/       # Global TypeScript types and interfaces
  utils/       # Shared utility functions and helper methods
```

### Anatomy of a Feature (`src/features/feature-name/`)
Each feature folder acts as a self-contained module:
*   `api/`: API request declarations (fetchers, mutations, react-query hooks).
*   `components/`: Components specific to this feature.
*   `hooks/`: Custom hooks for the feature's logic.
*   `routes/`: Route definitions and page components for the feature.
*   `stores/`: Localized state management (Zustand slices) for the feature.
*   `types/`: TypeScript types specific to the domain.
*   `index.ts`: The **Public API** of the feature. Export ONLY what other parts of the app need to access. Treat everything else as private.

*Rule*: Prevent cross-feature imports. Features should be composed at the application level (`src/app/` or `src/pages/`) rather than importing from each other directly.

---

## 2. Component Design Principles
*   Use **Functional Components** with Hooks.
*   **Single Responsibility**: Components should do one thing. Break down components > 150 lines.
*   **Props Destructuring**: Always destructure props in the function signature for immediate visibility.
*   **Dumb vs. Smart Components**: Keep UI components (dumb/presentational) in `src/components/`. Keep data-fetching and business logic (smart/container) in `src/features/`.
*   **Named Exports**: Prefer named exports for components to ensure consistent naming during imports and better refactoring support. Avoid `default export` except for lazy-loaded route components.

---

## 3. Styling Standards
*   **Vanilla CSS**: Use CSS Variables defined in `styles/variables.css` (or `index.css`) for theme switching (dark/light), typography, spacing, and animations.
*   **Tailwind CSS**: If using Tailwind, use `clsx` and `tailwind-merge` (`twMerge`) together (often combined in a `cn` utility function) to handle dynamic classes safely without CSS conflict.
*   **Aesthetics**: Always style with premium, rich visual design:
    *   Sleek glassmorphism (`backdrop-filter: blur(10px)`).
    *   Subtle micro-animations (`transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1)`).
    *   High-contrast, professional typography (e.g., `Inter`, `Outfit`).

---

## 4. State & Data Management
*   **Local State**: Use `useState` and `useReducer` for simple component-level state.
*   **Global UI State**: Use **Zustand** or Context API for lightweight global UI state (e.g., theme, sidebar open/close). Avoid over-engineering with Redux.
*   **Server State**: Use **TanStack Query (React Query)** or **SWR** for data fetching, caching, synchronization, and optimistic updates.
    *   *Rule*: Do NOT use `useEffect` for data fetching.
    *   Colocate query hooks inside `features/feature-name/api/`.

---

## 5. Performance Optimization & Vite Specifics
*   **Lazy Loading**: Use `React.lazy()` and `Suspense` for route-level code splitting to keep the initial bundle small.
*   **Memoization**: Use `useMemo` for expensive calculations and `useCallback` for functions passed as props to highly optimized child components. Do not overuse; only apply to fix measured bottlenecks.
*   **Vite Configuration**:
    *   Use `build.rollupOptions.output.manualChunks` for vendor chunk splitting.
    *   Set up path aliases (e.g., `@/components`) in `tsconfig.json` for cleaner imports.
*   **Asset Management**: Keep large images out of source assets; optimize and compile correctly using Vite assets.

---

## 6. Verification Workflow
*   `npm run build` — Verify production compilation and lack of TS/lint errors.
*   Check browser console for warning/error logs.
