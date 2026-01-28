# Expo Mobile Application Development Guide

This guide is created to provide context when working with Expo projects using Claude Code.

## Project Creation

```bash
# Create a new project
bunx create-expo -t default@next $ARGUMENTS
```

## Technology Stack

- **Framework**: Expo, React Native
- **Navigation**: Expo Router (file-based routing), NativeTabs
- **State Management**: React Context API
- **Translations**: i18next, react-i18next
- **Purchases**: RevenueCat (react-native-purchases)
- **Advertisements**: Google AdMob (react-native-google-mobile-ads)
- **Notifications**: expo-notifications
- **Animations**: react-native-reanimated
- **Storage**: localStorage via expo-sqlite polyfill

> **WARNING**: DO NOT USE AsyncStorage! Use expo-sqlite polyfill instead.

- Example usage

```js
import "expo-sqlite/localStorage/install";

globalThis.localStorage.setItem("key", "value");
console.log(globalThis.localStorage.getItem("key")); // 'value'
```

> **WARNING**: NEVER USE `lineHeight`! It causes layout issues in React Native. Use padding or margin instead.

## Project Structure

```
project-root/
├── src/
│   ├── app/
│   │   ├── _layout.tsx
│   │   ├── index.tsx
│   │   ├── explore.tsx
│   │   ├── settings.tsx
│   │   ├── paywall.tsx
│   │   └── onboarding.tsx
│   ├── components/
│   │   ├── ui/
│   │   ├── themed-text.tsx
│   │   └── themed-view.tsx
│   ├── constants/
│   │   ├── theme.ts
│   │   └── [data-files].ts
│   ├── context/
│   │   ├── onboarding-context.tsx
│   │   ├── theme-context.tsx
│   │   └── ads-context.tsx
│   ├── hooks/
│   │   ├── use-notifications.ts
│   │   ├── use-color-scheme.ts
│   │   └── use-interstitial-ad.ts
│   ├── lib/
│   │   ├── notifications.ts
│   │   ├── purchases.ts
│   │   ├── ads.ts
│   │   └── i18n.ts
│   └── locales/
│       ├── tr.json
│       └── en.json
├── assets/
│   └── images/
├── ios/
├── android/
├── app.json
├── eas.json
├── package.json
└── tsconfig.json
```

## Tab Navigation (NativeTabs)

Expo Router uses NativeTabs for native tab navigation:

```tsx
import { NativeTabs } from "expo-router/unstable-native-tabs";

export default function TabLayout() {
  return (
    <NativeTabs>
      <NativeTabs.Trigger name="index">
        <NativeTabs.Trigger.Label>Home</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="house.fill" md="home" />
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="explore">
        <NativeTabs.Trigger.Label>Explore</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="compass.fill" md="explore" />
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="settings">
        <NativeTabs.Trigger.Label>Settings</NativeTabs.Trigger.Label>
        <NativeTabs.Trigger.Icon sf="gear" md="settings" />
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```

### NativeTabs Properties

- **sf**: SF Symbols icon name (iOS)
- **md**: Material Design icon name (Android)
- **name**: Route file name
- Tab order follows trigger order

### Common Icons

| Purpose       | SF Symbol       | Material Icon |
| ------------- | --------------- | ------------- |
| Home          | house.fill      | home          |
| Explore       | compass.fill    | explore       |
| Settings      | gear            | settings      |
| Profile       | person.fill     | person        |
| Search        | magnifyingglass | search        |
| Favorites     | heart.fill      | favorite      |
| Notifications | bell.fill       | notifications |

## Development Commands

```bash
bun install
bun start
bun ios
bun android
bun lint
npx expo install --fix
npx expo prebuild --clean
```

## EAS Build Commands

```bash
eas build --profile development --platform ios
eas build --profile development --platform android
eas build --profile production --platform ios
eas build --profile production --platform android
eas submit --platform ios
eas submit --platform android
```

## Important Modules

### RevenueCat

- File: `lib/purchases.ts`
- Used for premium access
- Paywall: `app/paywall.tsx`

### AdMob

- Files: `src/lib/ads.ts`, `src/hooks/use-interstitial-ad.ts`
- Ads disabled for premium users
- Test IDs must be used in development

### Notifications

- Files: `src/lib/notifications.ts`, `src/hooks/use-notifications.ts`
- iOS requires push notification entitlement

### Onboarding

- File: `src/app/onboarding.tsx`
- Swipe-based screens
- Fullscreen background video
- Gradient overlay
- Redirects to paywall after completion

## Localization

- File: `lib/i18n.ts`
- Languages stored in `locales/`
- App restarts on language change

## Coding Standards

- Use functional components
- Strict TypeScript
- Avoid hardcoded strings
- Use padding instead of lineHeight
- Use memoization when necessary

## Context Providers

```tsx
<ThemeProvider>
  <OnboardingProvider>
    <AdsProvider>
      <Stack />
    </AdsProvider>
  </OnboardingProvider>
</ThemeProvider>
```

## Important Notes

1. iOS permissions are defined in `app.json`
2. Android permissions are defined in `app.json`
3. Enable new architecture via `newArchEnabled: true`
4. Enable typed routes via `experiments.typedRoutes`

## App Store & Play Store Notes

- iOS ATT permission required
- Restore purchases must work correctly
- Target SDK must be up to date

## Testing Checklist

- UI tested in all languages
- Dark / Light mode
- Notifications
- Premium flow
- Restore purchases
- Offline support
- Multiple screen sizes

## After Development

```bash
npx expo prebuild --clean
bun ios
bun android
```

> NOTE: `prebuild --clean` recreates ios and android folders. Run it after modifying native modules or app.json.
