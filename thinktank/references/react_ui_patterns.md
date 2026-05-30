# React UI Patterns Reference

This document outlines modern, high-leverage UI patterns for handling loading states, error handling, button feedback, and empty collections.

---

## 1. Loading State Patterns

### The Golden Rule: Avoid Loading Flashes
**Only show a loading indicator when there is no cached or existing data to display.** Do not show a full-screen loading spinner on background refetches or query refreshes, as this causes UI flashing and disrupts the user experience.

```typescript
// ✅ CORRECT: Only show loading screen when no data is available
const { data, isLoading, isError, refetch } = useQuery({ queryKey: ['items'], queryFn: fetchItems });

if (isError) return <ErrorState onRetry={refetch} />;
if (isLoading && !data) return <LoadingSkeleton />; // Show skeleton on initial load only
if (!data?.items.length) return <EmptyState />;

return <ItemList items={data.items} />;
```

```typescript
// ❌ WRONG: Shows full-screen loading on every refetch, causing UI flashing
if (isLoading) return <LoadingSpinner />;
```

### Loading Component Choice
*   **Skeleton Placeholders**: Use when the final layout shape is known (e.g., list items, cards, profiles). Skeletons reduce perceived loading time.
*   **Spinners / Activity Indicators**: Use for short, modal actions, inline operations, or form submissions where the layout shape is unknown.

---

## 2. Error Handling Patterns

### The Error Surface Hierarchy
1.  **Field-level Inline Errors**: For form inputs, validation feedback, and expected user input corrections.
2.  **Toast Notifications (Temporary)**: For recoverable, background, or action-specific failures (e.g., "Failed to save draft. Retrying...").
3.  **Page Banners (Persistent)**: For partial failures where some parts of the page are functional but others failed to load.
4.  **Full-Screen Error Fallbacks**: For unrecoverable routing or global app crashes, wrapping sections in an `ErrorBoundary`.

### Surfacing Errors (Never Swallow)
**CRITICAL**: Never catch errors silently without updating the user interface. Users must always receive feedback when an action fails.

```typescript
// ✅ CORRECT: Always log and toast errors to the user
const mutation = useMutation({
  mutationFn: createItem,
  onSuccess: () => {
    toast.success("Item created successfully");
  },
  onError: (error) => {
    console.error("Failed to create item:", error);
    toast.error(`Error: ${error.message || "Failed to create item"}`);
  }
});
```

---

## 3. Button State Patterns

### Disable and Spin on Submission
Always disable buttons and trigger visual loading indicators during async operations to prevent double-submissions or duplicate network requests.

```tsx
// ✅ CORRECT: Disabled and shows loading state during submission
<Button
  onClick={handleSubmit}
  disabled={isSubmitting || !isValid}
  isLoading={isSubmitting}
>
  Save Changes
</Button>
```

---

## 4. Empty State Requirements

Every dynamic list or grid collection must handle the scenario where the data array is empty. Present helpful descriptions, icons, and contextual call-to-actions.

```tsx
// ✅ CORRECT: Expo FlatList ListEmptyComponent pattern
return (
  <FlatList
    data={items}
    renderItem={({ item }) => <ItemRow item={item} />}
    ListEmptyComponent={
      <EmptyState
        icon="folder-open"
        title="No items found"
        description="Get started by creating your first item."
        action={{ label: "Add Item", onPress: handleAddItem }}
      />
    }
  />
);
```
