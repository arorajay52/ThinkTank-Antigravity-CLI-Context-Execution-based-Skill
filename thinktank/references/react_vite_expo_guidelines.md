# React Vite & React Expo: Technical Guidelines

This document provides verified, high-leverage architectural patterns and strategies for building applications using React Vite (web) and React Expo (mobile). **This is a mandatory reference** — read relevant sections before writing any frontend code.

---

## 1. React Vite (Web Frontend)

### 1.1 Feature-Based Architecture (Bulletproof React)
Organize code by **features** rather than by file type. This encapsulates related logic and makes the codebase highly scalable.
*   **Directory Structure**:
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
*   **Anatomy of a Feature (`src/features/feature-name/`)**:
    Each feature should be self-contained:
    *   `api/`: API request declarations (fetchers, mutations, react-query hooks)
    *   `components/`: Components specific to this feature
    *   `hooks/`: Custom hooks for the feature's logic
    *   `routes/`: Route definitions and page components for the feature
    *   `stores/`: Localized state management (Zustand slices)
    *   `types/`: Domain-specific TypeScript types
    *   `index.ts`: The **Public API** of the feature (export only what other parts of the app need).
    *   *Rule*: Prevent cross-feature imports. Features should be composed at the page/app level rather than importing from each other directly.

### 1.2 Component & Styling Standards
*   **Vanilla CSS & Tailwind**: 
    *   If using CSS: Use CSS Variables defined in `styles/variables.css` (or `index.css`) for theme switching (dark/light), typography, spacing, and animations.
    *   If using Tailwind: Use `clsx` and `tailwind-merge` (`twMerge`) together (often combined in a `cn` utility function) to handle dynamic classes safely without CSS conflict.
*   **Named Exports**: Prefer named exports for components to ensure consistent naming during imports and better refactoring support. Avoid `default export` except for lazy-loaded route components.
*   **Aesthetics**: Always style with premium, rich visual design:
    *   Sleek glassmorphism (`backdrop-filter: blur(10px)`).
    *   Subtle micro-animations (`transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1)`).
    *   High-contrast, professional typography (e.g., `Inter`, `Outfit`).

### 1.3 State & Data Management
*   **Local State**: Use `useState` and `useReducer` for simple component-level state.
*   **Global UI State**: Use **Zustand** or Context API for lightweight global UI state (e.g., theme, sidebar open/close). Avoid over-engineering with Redux.
*   **Server State**: Use **TanStack Query (React Query)** or **SWR** for data fetching, caching, synchronization, and optimistic updates.
    *   *Rule*: Do NOT use `useEffect` for data fetching.
    *   Colocate query hooks inside `features/feature-name/api/`.

### 1.4 Performance & Optimization
*   **Code Splitting (Lazy Loading)**: Use `React.lazy()` and `Suspense` for route-level code splitting to keep the initial bundle small.
*   **Memoization**: Use `useMemo` for expensive calculations and `useCallback` for functions passed as props to highly optimized child components. Do not overuse; only apply to fix measured bottlenecks.
*   **Asset Management**: Keep large images out of source assets; optimize and compile correctly using Vite assets.

---

## 2. React Expo (Mobile Applications)

### Directory & Navigation (Expo Router)
*   Use the file-system-based **Expo Router** (app directory) for clean navigation:
```
app/
├── (tabs)/          # Tab-based navigation group
│   ├── _layout.tsx  # Tabs definition (TabBar design)
│   ├── index.tsx    # Home tab
│   └── profile.tsx  # Profile tab
├── (auth)/          # Authentication flow group
├── _layout.tsx      # Root slot, providers, and global themes
└── modal.tsx        # Presentation sheet / modal
```

### 2.1 Layout & Overflow Prevention (CRITICAL)

These rules prevent the #1 most common mobile UI bug — content overflowing or getting cut off.

| Rule | Why |
|------|-----|
| Every screen root must be `<ScrollView>` or `<FlatList>` | Raw `<View>` cannot scroll — content gets cut off on small screens |
| Always set `flex: 1` on the outermost container | Without it, the container collapses to content height and may not fill the screen |
| Use `flexShrink: 1` on text inside horizontal rows | Prevents text from pushing other elements off-screen |
| Never use fixed `width` on text elements | Use `flex: 1` instead so text wraps naturally |
| Use `gap` property for spacing between children | Not margin hacks — `gap` is cleaner and doesn't cause collapse issues |
| Wrap long forms in `<KeyboardAvoidingView>` | Otherwise keyboard covers the inputs on both iOS and Android |

**Correct screen structure:**
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

**Common row layout (text + badge):**
```tsx
<View style={{ flexDirection: 'row', alignItems: 'center', gap: 8 }}>
  <Text style={{ flex: 1, flexShrink: 1 }} numberOfLines={1}>
    Long text that might overflow
  </Text>
  <View style={{ backgroundColor: '#333', borderRadius: 8, paddingHorizontal: 8 }}>
    <Text>Badge</Text>
  </View>
</View>
```

### 2.2 Responsive Sizing

| Pattern | Usage |
|---------|-------|
| `Dimensions.get('window').width` | For calculating card widths, grid layouts |
| Percentage-based `width: '90%'` | For main content containers |
| `minHeight` instead of `height` | For content that may expand (text, lists) |
| `maxWidth: 500` on web-targeted layouts | Prevents ultra-wide content on tablets |
| `allowFontScaling={false}` | For UI labels that must not resize with system font settings |

**Never hard-code pixel dimensions for layout containers.** Use `flex`, percentages, or calculated values.

### 2.3 Component Patterns

**Reusable Card:**
```tsx
const Card = ({ children, style }: { children: React.ReactNode; style?: ViewStyle }) => (
  <View style={[{
    backgroundColor: colors.cardBg,
    borderRadius: 12,
    padding: 16,
    // Android shadow
    elevation: 2,
    // iOS shadow
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.1,
    shadowRadius: 3,
  }, style]}>
    {children}
  </View>
);
```

**Reusable Confirm Modal (if not already in codebase):**
```tsx
const ConfirmModal = ({ visible, title, message, onConfirm, onCancel }) => (
  <Modal transparent visible={visible} animationType="fade">
    <View style={styles.overlay}>
      <View style={styles.dialog}>
        <Text style={styles.title}>{title}</Text>
        <Text style={styles.message}>{message}</Text>
        <View style={styles.buttonRow}>
          <TouchableOpacity onPress={onCancel}><Text>Cancel</Text></TouchableOpacity>
          <TouchableOpacity onPress={onConfirm}><Text>Confirm</Text></TouchableOpacity>
        </View>
      </View>
    </View>
  </Modal>
);
```

**Touch target minimums:**
*   All interactive elements must have at least **44px height** (Apple HIG) / **48dp** (Material)
*   Use `hitSlop={{ top: 8, bottom: 8, left: 8, right: 8 }}` for small icons

### 2.4 State & Data Flow

| Pattern | Rule |
|---------|------|
| Single source of truth | Lift state to App.tsx or Context — never duplicate state across components |
| Prop naming | Callbacks: `onAction` (not `handleAction` in prop types). Internal handlers: `handleAction` |
| Type safety | Always define `interface Props {}` — never use `any` for component props |
| `useCallback` | Wrap ALL functions passed as props to prevent child re-renders |
| `useMemo` | Use for expensive computations (filtering, sorting, calculating stats) |
| State updates | Always create NEW references: `setItems([...items, newItem])` not `items.push(newItem)` |
| Async state | Use `updateDataState` pattern: async wrapper that saves + sets state atomically |

**Anti-patterns to NEVER use:**
```tsx
// ❌ BAD: Inline function in prop — causes re-render every time
<Child onPress={() => doSomething(id)} />

// ✅ GOOD: useCallback
const handlePress = useCallback(() => doSomething(id), [id]);
<Child onPress={handlePress} />

// ❌ BAD: Mutating state directly
data.items.push(newItem);
setData(data);

// ✅ GOOD: New reference
setData({ ...data, items: [...data.items, newItem] });
```

### 2.5 Performance Rules

| Rule | Why |
|------|-----|
| **Mount ALL tabs persistently** | Use `display: activeTab === 'x' ? 'flex' : 'none'` — never `{activeTab === 'x' && <Tab />}` |
| **Use `FlatList`** for lists > 20 items | `.map()` in `<ScrollView>` renders everything at once — bad for memory |
| **Use `StyleSheet.create()`** | Inline style objects `{{ color: 'red' }}` create new objects every render |
| **Avoid anonymous functions in render** | Wrap in `useCallback` or define outside component |
| **Use `React.memo`** for pure display components | Prevents re-render when parent state changes but props don't |
| **Debounce search inputs** | Never filter on every keystroke — use 300ms debounce |

**Tab mounting pattern (CANONICAL):**
```tsx
// ✅ Persistent mounting — instant tab switches, no state loss
<View style={{ flex: 1, display: activeTab === 'home' ? 'flex' : 'none' }}>
  <HomeView />
</View>
<View style={{ flex: 1, display: activeTab === 'profile' ? 'flex' : 'none' }}>
  <ProfileView />
</View>
```

### 2.6 Visual Polish Checklist

Before declaring any UI work complete, verify:

- [ ] **Border radius**: Consistent scale — `8` for small elements, `12` for cards, `20-24` for buttons/pills
- [ ] **Spacing scale**: Use multiples of 4: `4, 8, 12, 16, 20, 24, 32`
- [ ] **Shadows**: Cards have elevation (Android) + shadowOffset (iOS)
- [ ] **Touch feedback**: All `TouchableOpacity` have `activeOpacity={0.7}`
- [ ] **Loading states**: Async operations show feedback (spinner, skeleton, disabled button)
- [ ] **Empty states**: Lists show a message when empty (not just blank space)
- [ ] **Text truncation**: Long text uses `numberOfLines` + `ellipsizeMode="tail"`
- [ ] **Safe area**: Root screen uses `SafeAreaView` from `react-native-safe-area-context`
- [ ] **Keyboard**: Forms use `KeyboardAvoidingView` and `keyboardShouldPersistTaps="handled"`
- [ ] **Color consistency**: All colors come from a theme/design tokens, not hardcoded hex values

### Native Platform Constraints
*   **SafeAreaView**: Always wrap root screens with `SafeAreaView` from `react-native-safe-area-context` to avoid layout overlapping with status bars or home indicators.
*   **Images**: Use `expo-image` instead of React Native's standard `<Image>` to enable memory caching, smooth transitions, and high-performance rendering.
*   **Keyboard Handling**: Wrap input forms in `KeyboardAvoidingView` with `behavior={Platform.OS === 'ios' ? 'padding' : 'height'}` to ensure text inputs remain visible during keyboard interactions.
*   **Platform Branching**: Keep code clean. Use `Platform.select` or file suffixes (e.g., `Button.ios.tsx`, `Button.android.tsx`) for platform-specific styling or logic.

---

## 3. General Verification Workflow

Before declaring any feature complete, run these local verification commands:
*   **React Vite**:
    *   `npm run build` — Verify production compilation and lack of TS/lint errors.
    *   Check console for warning/error logs.
*   **React Expo**:
    *   `npx expo lint` (or equivalent eslint) to find static analysis errors.
    *   `node node_modules/typescript/lib/tsc.js --noEmit` to verify type safety.
    *   Test on smallest supported screen size (320px width) to catch overflow issues.
