# Expo Mobile Deployment Reference

This document outlines best practices for production builds, app store submissions, and over-the-air (OTA) updates for Expo and React Native applications.

---

## 1. Pre-Deployment Optimization Check

Before building production binaries, perform these optimization checks:
- **Dependency Audit**: Ensure there are no unused dependencies, and all packages match the corresponding React Native version (`npx expo install --check`).
- **Bundle Optimization**: Ensure assets (images, icons, fonts) are minimized and compressed to reduce initial download size.
- **Environment Variables**: Configure your production variables securely in `app.json` or through EAS secret environment variables (never commit raw production keys).
- **Static Analysis & Type Checking**: Ensure lint and TypeScript checks pass clean:
  *   `npx expo lint` (or project ESLint command)
  *   `tsc --noEmit` (or typescript check)

---

## 2. Production Builds via EAS Build

Always use **Expo Application Services (EAS)** for production iOS and Android builds. EAS runs builds in isolated, clean cloud environments, eliminating local machine compilation discrepancies.

### Common Commands:
*   Configure EAS: `eas build:configure`
*   Build for Android (Production): `eas build --platform android --profile production`
*   Build for iOS (Production): `eas build --platform ios --profile production`
*   Build for all platforms: `eas build --platform all`

---

## 3. App Store Submissions (EAS Submit)

Automate submission of completed native binaries directly to Google Play and App Store Connect using EAS Submit.

### Configuration (`eas.json`):
```json
{
  "submit": {
    "production": {
      "android": {
        "serviceAccountKeyPath": "./pc-api-key.json",
        "track": "production"
      },
      "ios": {
        "appleId": "developer@company.com",
        "ascAppId": "123456789",
        "appleTeamId": "AB12CD34EF"
      }
    }
  }
}
```

### Commands:
*   Submit a specific build: `eas submit --platform all` (you can choose build IDs interactively).
*   Build and submit automatically: `eas build --platform all --auto-submit`

---

## 4. Over-The-Air (OTA) Updates (`expo-updates`)

Use over-the-air updates to fix critical bugs or deliver content changes directly to users' devices without waiting for app store review processes.

### Runtime Update Flow:
*   Updates are scoped by **Runtime Version** (configured in `app.json`). If the native codebase changes (e.g. adding a new native library or upgrading Expo SDK), you **must** bump the native runtime version and submit a new app store binary.
*   JS-only updates (changing styles, UI text, business logic) can be pushed instantly as updates to the same runtime version.

### Configuration (`app.json`):
```json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/your-project-id",
      "enabled": true,
      "fallbackToCacheTimeout": 0
    },
    "runtimeVersion": {
      "policy": "appVersion"
    }
  }
}
```

### Commands:
*   Configure Updates: `eas update:configure`
*   Publish update: `eas update --branch production --message "Fix critical dashboard crash"`
