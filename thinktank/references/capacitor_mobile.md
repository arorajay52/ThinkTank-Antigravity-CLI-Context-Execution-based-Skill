# Capacitor Mobile Guidelines

This document outlines best practices for wrapping web applications for mobile devices using **Capacitor.js**. Consult this file **only** when working on Capacitor mobile web wrapper features.

---

## 1. Viewport & Styling (Safe Area Insets)

Unlike native Expo apps, Capacitor runs inside a native web container (WebView). To prevent UI elements (headers, footers) from being hidden under device notches or home indicators, use CSS safe area environment variables:

```css
/* Add to global index.css or styling root */
body {
  padding-top: env(safe-area-inset-top, 0px);
  padding-bottom: env(safe-area-inset-bottom, 0px);
  padding-left: env(safe-area-inset-left, 0px);
  padding-right: env(safe-area-inset-right, 0px);
}
```

Ensure headers or navigation bars use `position: sticky` or absolute padding with safe area variables:
```css
.app-header {
  padding-top: max(12px, env(safe-area-inset-top));
}
```

---

## 2. Keyboard & Viewport Resizing

*   On Android, the soft keyboard resizes the web viewport by default, which can compress layout heights.
*   On iOS, the keyboard overlays the viewport.
*   Use the `@capacitor/keyboard` plugin to listen to keyboard events and adjust scroll offsets dynamically:
    ```typescript
    import { Keyboard } from '@capacitor/keyboard';

    Keyboard.addListener('keyboardWillShow', info => {
      console.log('Keyboard will show with height:', info.keyboardHeight);
    });
    ```

---

## 3. Capacitor Native Plugins

Avoid using direct browser APIs for device capabilities when wrapped. Use Capacitor plugins for better native device integration:

| Resource | Web API | Capacitor Plugin |
|----------|---------|------------------|
| Storage | `localStorage` | `@capacitor/preferences` (Prevents OS-level data eviction) |
| Dialogs | `alert()`, `confirm()` | `@capacitor/dialog` (Provides native modal alerts) |
| Connectivity | `navigator.onLine` | `@capacitor/network` (Provides accurate link status) |
| Haptics | N/A | `@capacitor/haptics` (Trigger vibrations on touch feedback) |

---

## 4. Build & Sync Workflow

Every code change must be compiled to static web assets and synced to native platforms:

1.  **Build Web App**: `npm run build` (creates production folder, e.g. `dist/` or `build/`).
2.  **Sync Web Assets to Native Native Platforms**: `npx cap sync` (copies assets to android/ios folders and syncs plugins).
3.  **Run Native Client**:
    *   Android Studio: `npx cap open android`
    *   Xcode: `npx cap open ios`
