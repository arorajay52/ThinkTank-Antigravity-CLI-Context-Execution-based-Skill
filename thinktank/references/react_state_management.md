# React State Management Reference

This document details guidelines for separating global UI state (client-side) from remote resource state (server-side), implementing optimistic UI updates, and organizing TypeScript state slices.

---

## 1. Global UI State vs. Server State

| Category | Description | Primary Solution |
|----------|-------------|------------------|
| **Client State** | Ephemeral UI states (theme, sidebars, active modals, temporary filters) | **Zustand** or React Context |
| **Server State** | Remote database resources (users, orders, cached api results, pagination) | **TanStack Query (React Query)** or SWR |

### The Core Don't: Do Not Duplicate Server State
Do not pull fetched server data into a local `useState` or global Zustand store for editing unless you are building a complex offline form. Let TanStack Query manage cache invalidation, polling, and synchronization.

---

## 2. Zustand Slices Pattern (Scalable UI State)

For larger apps, split the global Zustand store into separate, domain-focused slices.

```typescript
import { create, StateCreator } from 'zustand';

// 1. Define slices and their types
interface UISlice {
  sidebarOpen: boolean;
  toggleSidebar: () => void;
}

interface UserSlice {
  currentUser: User | null;
  setCurrentUser: (user: User | null) => void;
}

const createUISlice: StateCreator<UISlice & UserSlice, [], [], UISlice> = (set) => ({
  sidebarOpen: true,
  toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
});

const createUserSlice: StateCreator<UISlice & UserSlice, [], [], UserSlice> = (set) => ({
  currentUser: null,
  setCurrentUser: (currentUser) => set({ currentUser }),
});

// 2. Combine slices into a single unified store
export const useStore = create<UISlice & UserSlice>()((...args) => ({
  ...createUISlice(...args),
  ...createUserSlice(...args),
}));

// 3. Create selector hooks to prevent unnecessary re-renders
export const useSidebarOpen = () => useStore((state) => state.sidebarOpen);
export const useToggleSidebar = () => useStore((state) => state.toggleSidebar);
export const useCurrentUser = () => useStore((state) => state.currentUser);
```

---

## 3. TanStack Query Keys Factory Pattern

Always organize query keys using a structured factory pattern. This ensures consistent cache access and prevents query key typos.

```typescript
export const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: string) => [...todoKeys.details(), id] as const,
};

// Usage in query hook
export function useTodo(id: string) {
  return useQuery({
    queryKey: todoKeys.detail(id),
    queryFn: () => fetchTodoById(id),
    enabled: !!id,
  });
}
```

---

## 4. Optimistic Updates Pattern

Optimistic updates make the application feel instantaneous by rendering changes before the network request resolves, with automatic rollback if the request fails.

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';

export function useUpdateTodo() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateTodoApi,
    // Step 1: When mutate is called
    onMutate: async (updatedTodo) => {
      // Cancel outgoing refetches so they don't overwrite our optimistic update
      await queryClient.cancelQueries({ queryKey: todoKeys.detail(updatedTodo.id) });

      // Snapshot the previous todo value
      const previousTodo = queryClient.getQueryData(todoKeys.detail(updatedTodo.id));

      // Optimistically update the cache to the new value
      queryClient.setQueryData(todoKeys.detail(updatedTodo.id), updatedTodo);

      // Return context containing previous value for rollback
      return { previousTodo };
    },
    // Step 2: If the mutation fails
    onError: (err, updatedTodo, context) => {
      if (context?.previousTodo) {
        queryClient.setQueryData(todoKeys.detail(updatedTodo.id), context.previousTodo);
      }
    },
    // Step 3: Always refetch/invalidate after success or failure
    onSettled: (data, error, variables) => {
      queryClient.invalidateQueries({ queryKey: todoKeys.detail(variables.id) });
    },
  });
}
```
