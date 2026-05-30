# React Expo (Native Mobile) Technical Guidelines

This document outlines verified, high-leverage architectural patterns and guidelines for building native mobile applications using React Expo. Consult this file **only** when working on native mobile-targeted features.

---

## 1. Directory & Navigation (Expo Router)

Use the file-system-based **Expo Router** (app directory) for clean navigation:
```text
app/
├── (tabs)/          # Tab-based navigation group
│   ├── _layout.tsx  # Tabs definition (TabBar design)
│   ├── index.tsx    # Home tab
│   └── profile.tsx  # Profile tab
├── (auth)/          # Authentication flow group
├── _layout.tsx      # Root slot, providers, and global themes
└── modal.tsx        # Presentation sheet / modal
```

---

## 2. Layout & Overflow Prevention (CRITICAL)

These rules prevent the #1 most common mobile UI bug — content overflowing or getting cut off.

| Rule | Why |
|------|-----|
| Every screen root must be `<ScrollView>` or `<FlatList>` | Raw `<View>` cannot scroll — content gets cut off on small screens |
| Always set `flex: 1` on the outermost container | Without it, the container collapses to content height and may not fill the screen |
| Use `flexShrink: 1` on text inside horizontal rows | Prevents text from pushing other elements off-screen |
| Never use fixed `width` on text elements | Use `flex: 1` instead so text wraps naturally |
| Use `gap` property for spacing between children | Not margin hacks — `gap` is cleaner and doesn't cause collapse issues |
| Wrap long forms in `<KeyboardAvoidingView>` | Otherwise keyboard covers the inputs on both iOS and Android |

### Correct Screen Structure:
```tsx
<SafeAreaView style={{ flex: 1, backgroundColor: colors.bg }}>
  <ScrollView
    style={{ flex: 1 }}
    contentContainerStyle={{ padding: 16, gap: 16 }}
    keyboardShouldPersistTaps="handled"
  >
    {/* Screen content here */}
  </ScrollView>
</SafeAreaView>
```

---

## 3. Responsive Sizing
*   Use `useWindowDimensions` over `Dimensions.get()` to calculate dynamic widths or responsive column scales.
*   Use percentage-based sizes (e.g. `width: '90%'`) for content blocks.
*   Set `minHeight` instead of fixed `height` for expandable text containers or blocks.
*   Set `maxWidth: 500` on layouts designed to scale well on tablets.
*   Set `allowFontScaling={false}` on fixed UI labels to prevent text breaking when system font sizing is scaled up.

---

## 4. Component & Styling Patterns
*   **Touch Target Minimums**: All interactive elements must have at least **44px height** (Apple HIG) / **48dp** (Material). Use `hitSlop={{ top: 8, bottom: 8, left: 8, right: 8 }}` for small icons.
*   **Shadows**: Use CSS `boxShadow` style prop. NEVER use legacy React Native shadow or elevation styles.
    ```tsx
    <View style={{ boxShadow: "0 1px 2px rgba(0, 0, 0, 0.05)" }} />
    ```
*   **Immutability**: Always treat state updates as immutable (use spread operators, never push values directly).
*   **Optimization**: Wrap functions passed down as props in `useCallback` to prevent child component re-renders.

---

## 5. Performance Rules
*   **Mount ALL Tabs Persistently**: Use `display: activeTab === 'x' ? 'flex' : 'none'` rather than `{activeTab === 'x' && <Tab />}` to prevent mounting/unmounting lag.
*   **List Virtualization**: Use `FlatList` or `SectionList` for collections with $>20$ items to prevent memory bloat.
*   **Avoid Inline Styles inside Maps/Loops**: Inline style objects `{{ color: 'red' }}` create new objects on every render. Use `StyleSheet.create` for loop items.

---

## 6. Verification Workflow
*   `tsc --noEmit` — verify TypeScript type-safety.
*   `npx expo lint` — run static analysis linter.
*   Test layouts on small screen widths (e.g., 320px) to verify overflow prevention.
