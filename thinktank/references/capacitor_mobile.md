# Capacitor Mobile Guidelines (Verified Best Practices)

This document outlines official best practices from the Ionic/Capacitor core team for building production-ready hybrid mobile applications.

---

## 1. Encapsulate Capacitor Plugins (Wrapper Services)

**Never use Capacitor plugins directly inside your component views.** Instead, create dedicated wrapper services or provider hooks to encapsulate plugin calls.

*   *Why*: Centralizes platform availability checks, simplifies mocking native interfaces during unit testing, and ensures API upgrades only require changes in a single file.

### Example Plugin Wrapper:
```typescript
// services/storage-service.ts
import { Preferences } from '@capacitor/preferences';

export const storageService = {
  async get(key: string): Promise<string | null> {
    try {
      const { value } = await Preferences.get({ key });
      return value;
    } catch (error) {
      console.error('Capacitor Preferences get error:', error);
      return null;
    }
  },
  async set(key: string, value: string): Promise<void> {
    await Preferences.set({ key, value });
  }
};
```

---

## 2. Treat Native Folders as Source Code

The `android` and `ios` platforms directories created by Capacitor **are not temporary files or build artifacts**. 
*   **Version Control**: Commit these directories to Git.
*   **Direct Modification**: Perform all platform configurations (modifying plist variables in `Info.plist`, editing application permissions in `AndroidManifest.xml`, or adding custom Gradle hooks) directly inside their respective folders.

---

## 3. WebView Security & Secure Storage

*   **Secrets Prevention**: Never embed API secret keys or private credentials in raw frontend code or asset bundles, as they can be easily unpacked from the WebView bundle. Keep sensitive computation on the server-side.
*   **Secure Storage**: Do not use standard `localStorage` or `sessionStorage` for storing authentication tokens or PII, as WebViews can evict this data on memory pressure. Use secure storage wrapper plugins that bind to **iOS Keychain** and **Android Keystore** (e.g. `@capawesome/capacitor-secure-storage` or `@ionic/secure-storage`).
*   **Safe Area Handling**: Ensure headers and interactive buttons account for device notches and home indicators using CSS safe-area variables:
    ```css
    body {
      padding-top: env(safe-area-inset-top, 0px);
      padding-bottom: env(safe-area-inset-bottom, 0px);
    }
    ```

---

## 4. Development & Build Steps

1.  **Prioritize Official Plugins**: Use official `@capacitor/*` core plugins over outdated Cordova plugins for stability and native compilation speed.
2.  **Web Assets Compilation**: Compile the web app to build files (e.g. `dist/` or `build/` directory).
3.  **Syncing to Native**: Run `npx cap sync` to copy assets and update native bindings.
4.  **Testing**: Always test native device APIs (camera, push notifications, biometrics) on physical iOS and Android hardware early in the process.
