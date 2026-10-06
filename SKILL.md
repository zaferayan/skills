---
name: zafer-skills
description: Expo React Native mobile app development with StoreKit 2 subscriptions (expo-iap) plus RevenueCat observer mode, AdMob ads with UMP/ATT consent, PostHog analytics, i18n with RTL, onboarding flow, paywall, and NativeTabs navigation
---

# Expo Mobile Application Development Guide

> **IMPORTANT**: This is a SKILL file, NOT a project. NEVER run npm/bun install in this folder. NEVER create code files here. When creating a new project, ALWAYS ask the user for the project path first or create it in a separate directory (e.g., `~/Projects/app-name`).

This guide is created to provide context when working with Expo projects using Claude Code.

## MANDATORY REQUIREMENTS

When creating a new Expo project, you MUST include ALL of the following:

### Required Screens (ALWAYS CREATE)

- [ ] `src/app/onboarding.tsx` - Swipe-based onboarding with fullscreen background video and gradient overlay
- [ ] `src/app/notification-permission.tsx` - Pre-permission screen that explains notifications before the system prompt (add `location-permission.tsx` too if the app uses location)
- [ ] `src/app/paywall.tsx` - Paywall screen (shown after the permission screens)
- [ ] `src/app/settings.tsx` - Settings screen with language, theme, notifications, and reset onboarding options

### Onboarding Video Implementation (REQUIRED)

The onboarding screen MUST have a fullscreen background video. Use a URL, not a local file:

```tsx
import { useVideoPlayer, VideoView } from "expo-video";

const VIDEO_URL =
  "https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4";

const player = useVideoPlayer(VIDEO_URL, (player) => {
  player.loop = true;
  player.muted = true;
  player.play();
});

// In render:
<VideoView
  player={player}
  style={StyleSheet.absoluteFill}
  contentFit="cover"
  nativeControls={false}
/>;
```

Do NOT just import expo-video without actually using the VideoView component.

### Required Navigation (ALWAYS USE)

- [ ] Use `NativeTabs` from `expo-router/unstable-native-tabs` for tab navigation - NEVER use `@react-navigation/bottom-tabs` or `Tabs` from expo-router

### Required Context Providers (ALWAYS WRAP)

```tsx
import { ThemeProvider } from "@/context/theme-context";
import {
  DarkTheme,
  DefaultTheme,
  ThemeProvider as NavigationThemeProvider,
} from "@react-navigation/native";

<ThemeProvider>
  <OnboardingProvider>
    <AdsProvider>
      <NavigationThemeProvider
        value={colorScheme === "dark" ? DarkTheme : DefaultTheme}
      >
        <Stack />
      </NavigationThemeProvider>
    </AdsProvider>
  </OnboardingProvider>
</ThemeProvider>;
```

`AdsProvider` must be inside `OnboardingProvider`: it waits for onboarding to finish before showing the consent form / ATT prompt (see Ads Consent).

### Required Libraries (ALWAYS INSTALL)

Use `npx expo install` to install libraries (NOT npm/yarn/bun install):

```bash
npx expo install expo-iap react-native-purchases react-native-google-mobile-ads expo-tracking-transparency expo-notifications i18next react-i18next expo-localization react-native-reanimated expo-video expo-audio expo-sqlite expo-linear-gradient expo-store-review expo-updates expo-dev-client posthog-react-native expo-haptics
```

Libraries:

- `expo-iap` (StoreKit 2 / Play Billing - purchases and premium check)
- `react-native-purchases` (RevenueCat - observer mode only: purchase records + Apple Ads attribution)
- `react-native-google-mobile-ads` (AdMob + Google UMP consent)
- `expo-tracking-transparency` (iOS ATT prompt)
- `posthog-react-native` (analytics, error tracking, feature flags)
- `expo-store-review` (native rating prompt)
- `expo-updates` (reload after RTL/LTR change), `expo-dev-client`
- `expo-haptics`
- `expo-notifications`
- `i18next` + `react-i18next` + `expo-localization`
- `react-native-reanimated`
- `expo-video` + `expo-audio`
- `expo-sqlite` (for localStorage)
- `expo-linear-gradient` (for gradient overlays)

### AdMob Configuration (REQUIRED in app.json)

You MUST add this to `app.json` for AdMob to work:

```json
{
  "expo": {
    "plugins": [
      [
        "react-native-google-mobile-ads",
        {
          "androidAppId": "ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy",
          "iosAppId": "ca-app-pub-xxxxxxxxxxxxxxxx~yyyyyyyyyy"
        }
      ]
    ]
  }
}
```

For development/testing, use test App IDs:

- iOS: `ca-app-pub-3940256099942544~1458002511`
- Android: `ca-app-pub-3940256099942544~3347511713`

Do NOT skip this configuration or the app will crash with `GADInvalidInitializationException`.

Also add these plugins (ATT text is mandatory on iOS when ads can be personalised):

```json
[
  "expo-tracking-transparency",
  {
    "userTrackingPermission": "Your permission lets us show ads that are more relevant to you. If you decline, you can keep using the app and will see less relevant ads."
  }
],
"expo-iap"
```

### Banner Ad Implementation (REQUIRED)

Put the banner in its own component and float it above the tab bar in the Tab layout:

```tsx
// src/components/banner-ad.tsx
import { useState } from 'react';
import { View, StyleSheet } from 'react-native';
import { BannerAd, BannerAdSize, TestIds } from 'react-native-google-mobile-ads';
import { useAds } from '@/context/ads-context';
import { AD_UNIT_IDS } from '@/lib/ads';

export function AdBanner() {
  const { isAdsEnabled } = useAds();
  const [loaded, setLoaded] = useState(false);
  if (!isAdsEnabled) return null;

  return (
    // Hidden until filled, so an empty slot never takes space
    <View style={[styles.container, !loaded && styles.hidden]}>
      <BannerAd
        unitId={__DEV__ ? TestIds.BANNER : AD_UNIT_IDS.BANNER ?? TestIds.BANNER}
        size={BannerAdSize.ANCHORED_ADAPTIVE_BANNER}
        // Personalisation follows the UMP + ATT choice; do NOT force requestNonPersonalizedAdsOnly
        onAdLoaded={() => setLoaded(true)}
        onAdFailedToLoad={() => setLoaded(false)}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { alignItems: 'center', width: '100%' },
  hidden: { opacity: 0, height: 0 },
});
```

```tsx
// src/app/(tabs)/_layout.tsx
import { I18nManager, Platform, StyleSheet, View } from 'react-native';
import { usePathname } from 'expo-router';
import { NativeTabs } from 'expo-router/unstable-native-tabs';
import { useSafeAreaInsets } from 'react-native-safe-area-context';
import { useTranslation } from 'react-i18next';
import { AdBanner } from '@/components/banner-ad';

const TAB_BAR_HEIGHT = Platform.OS === 'ios' ? 49 : 56;

export default function TabLayout() {
  const { t } = useTranslation();
  const insets = useSafeAreaInsets();
  // Hide the banner on screens where it would get in the way (camera, compass, full-screen media...)
  const showBanner = usePathname() !== '/camera';

  return (
    <View style={styles.container}>
      <NativeTabs
        // The native tab bar ignores React Native's RTL setting
        unstable_nativeProps={{ direction: I18nManager.isRTL ? 'rtl' : 'ltr' }}
      >
        <NativeTabs.Trigger name="index">
          <NativeTabs.Trigger.Label>{t('tabs.home')}</NativeTabs.Trigger.Label>
          <NativeTabs.Trigger.Icon sf="house.fill" md="home" />
        </NativeTabs.Trigger>
        <NativeTabs.Trigger name="settings">
          <NativeTabs.Trigger.Label>{t('tabs.settings')}</NativeTabs.Trigger.Label>
          <NativeTabs.Trigger.Icon sf="gear" md="settings" />
        </NativeTabs.Trigger>
      </NativeTabs>

      {showBanner && (
        <View style={[styles.banner, { bottom: TAB_BAR_HEIGHT + insets.bottom }]}>
          <AdBanner />
        </View>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1 },
  banner: { position: 'absolute', left: 0, right: 0, alignItems: 'center' },
});
```

- ALWAYS use `TestIds.BANNER` in development
- Banner floats right above the tab bar (`TAB_BAR_HEIGHT + insets.bottom`); give scroll content enough bottom padding
- `useAds().isAdsEnabled` is false for premium users and until consent + SDK init are done

### Ads Consent: UMP + ATT (REQUIRED)

Ads must not be initialised before consent. Order: **Google UMP form (EU/UK GDPR) → iOS ATT prompt → `mobileAds().initialize()`**. Run it only after onboarding is finished, so it does not stack on top of the permission screens, and skip it entirely for premium users.

```ts
// src/lib/ads-consent.ts
import { Platform } from 'react-native';
import { AdsConsent, AdsConsentDebugGeography, AdsConsentPrivacyOptionsRequirementStatus } from 'react-native-google-mobile-ads';
import * as TrackingTransparency from 'expo-tracking-transparency';

// Test the EU form from anywhere: EXPO_PUBLIC_ADS_DEBUG_EEA=1 bun start (simulator / test devices only)
const DEBUG_EEA = __DEV__ && process.env.EXPO_PUBLIC_ADS_DEBUG_EEA === '1';

export async function gatherAdsConsent() {
  let canRequestAds = false;
  let privacyOptionsRequired = false;
  try {
    const info = await AdsConsent.gatherConsent(DEBUG_EEA ? { debugGeography: AdsConsentDebugGeography.EEA } : undefined);
    canRequestAds = info.canRequestAds;
    privacyOptionsRequired = info.privacyOptionsRequirementStatus === AdsConsentPrivacyOptionsRequirementStatus.REQUIRED;
  } catch {
    // Offline: fall back to the previous session's decision
    canRequestAds = (await AdsConsent.getConsentInfo().catch(() => null))?.canRequestAds ?? false;
  }
  // ATT only if ads will be shown; declining still shows (non-personalised) ads
  if (canRequestAds && Platform.OS === 'ios' && TrackingTransparency.isAvailable()) {
    const { status } = await TrackingTransparency.getTrackingPermissionsAsync();
    if (status === 'undetermined') await TrackingTransparency.requestTrackingPermissionsAsync();
  }
  return { canRequestAds, privacyOptionsRequired };
}

export const showAdsPrivacyOptions = () => AdsConsent.showPrivacyOptionsForm();
```

In `AdsProvider`:

- Check premium first (cached value, then store) - premium users never see the consent form
- After `gatherAdsConsent()` returns `canRequestAds`, call `mobileAds().setRequestConfiguration({ maxAdContentRating: MaxAdContentRating.PG, ... })` then `mobileAds().initialize()`
- Expose `isAdsEnabled`, `isPremium`, `privacyOptionsRequired`, `showPrivacyOptions`, `refreshPremiumStatus`
- Keep ads OFF in development by default (`__DEV__ && process.env.EXPO_PUBLIC_ADS_DEV !== '1'`)
- Settings MUST show a "Privacy choices" row when `privacyOptionsRequired` is true (EU rule)
- The consent form content is configured in AdMob console → Privacy & messaging

### TURKISH LOCALIZATION (IMPORTANT)

When writing `tr.json`, you MUST use correct Turkish characters:

- ı (lowercase dotless i) - NOT i
- İ (uppercase dotted I) - NOT I
- ü, Ü, ö, Ö, ç, Ç, ş, Ş, ğ, Ğ

Example:

- ✅ "Ayarlar", "Giriş", "Çıkış", "Başla", "İleri", "Güncelle"
- ❌ "Ayarlar", "Giris", "Cikis", "Basla", "Ileri", "Guncelle"

### FORBIDDEN (NEVER USE)

- ❌ AsyncStorage - Use `expo-sqlite/localStorage/install` instead
- ❌ lineHeight style - Use padding/margin instead
- ❌ `Tabs` from expo-router - Use `NativeTabs` instead
- ❌ `@react-navigation/bottom-tabs` - Use `NativeTabs` instead
- ❌ `expo-av` - Use `expo-video` for video, `expo-audio` for audio instead
- ❌ `expo-ads-admob` - Use `react-native-google-mobile-ads` instead
- ❌ Any other ads library - ONLY use `react-native-google-mobile-ads`
- ❌ Reanimated hooks inside callbacks - Call at component top level
- ❌ Hardcoded prices or discount badges on the paywall - Always use store-formatted prices (`displayPrice`)
- ❌ Calling `mobileAds().initialize()` before UMP consent
- ❌ Unsupervised `react-native-purchases` purchase calls (`purchasePackage`) - RevenueCat runs in observer mode, purchases go through `expo-iap`

### Reanimated Usage (IMPORTANT)

NEVER call `useAnimatedStyle`, `useSharedValue`, or other reanimated hooks inside callbacks, loops, or conditions.

❌ WRONG:

```tsx
const renderItem = () => {
  const animatedStyle = useAnimatedStyle(() => ({ opacity: 1 })); // ERROR!
  return <Animated.View style={animatedStyle} />;
};
```

✅ CORRECT:

```tsx
function MyComponent() {
  const animatedStyle = useAnimatedStyle(() => ({ opacity: 1 })); // Top level
  return <Animated.View style={animatedStyle} />;
}
```

For lists, create a separate component for each item:

```tsx
function AnimatedItem({ item }) {
  const animatedStyle = useAnimatedStyle(() => ({ opacity: 1 }));
  return <Animated.View style={animatedStyle}>{item.name}</Animated.View>;
}

// In FlatList:
renderItem={({ item }) => <AnimatedItem item={item} />}
```

### POST-CREATION CLEANUP (ALWAYS DO)

After creating a new Expo project, you MUST:

1. If using `(tabs)` folder, DELETE `src/app/index.tsx` to avoid route conflicts:

```bash
rm src/app/index.tsx
```

2. Check and remove `lineHeight` from these files:

- `src/components/themed-text.tsx` (comes with lineHeight by default - REMOVE IT)
- Any other component using `lineHeight`

Search and remove all `lineHeight` occurrences:

```bash
grep -r "lineHeight" src/
```

Replace with padding or margin instead.

### AFTER COMPLETING CODE (ALWAYS RUN)

When you finish writing/modifying code, you MUST run these commands in order:

```bash
npx expo install --fix
npx expo prebuild --clean
```

1. `install --fix` fixes dependency version mismatches
2. `prebuild --clean` recreates ios and android folders

Do NOT skip these steps.

---

## Project Creation

When user asks to create an app, you MUST:

1. FIRST ask for the bundle ID (e.g., "What is the bundle ID? Example: com.company.appname")
2. Create the project in the CURRENT directory using:

```bash
bunx create-expo -t default@next app-name
```

The current baseline is Expo SDK 57 / React Native 0.86 / React 19.2 (New Architecture only). Enable the React Compiler and typed routes in `app.json`:

```json
{
  "expo": {
    "experiments": { "typedRoutes": true, "reactCompiler": true },
    "ios": {
      "infoPlist": {
        "ITSAppUsesNonExemptEncryption": false,
        "CFBundleAllowMixedLocalizations": true
      }
    }
  }
}
```

3. Update `app.json` with the bundle ID:

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.company.appname"
    },
    "android": {
      "package": "com.company.appname"
    }
  }
}
```

4. Then cd into the project and start implementing all required screens
5. Do NOT ask for project path - always use current directory

## Technology Stack

- **Framework**: Expo, React Native
- **Navigation**: Expo Router (file-based routing), NativeTabs
- **State Management**: React Context API
- **Translations**: i18next, react-i18next
- **Purchases**: StoreKit 2 / Play Billing via `expo-iap` (source of truth for premium)
- **Purchase records & Apple Ads attribution**: RevenueCat in observer mode (`react-native-purchases`)
- **Advertisements**: Google AdMob (react-native-google-mobile-ads) + UMP consent + ATT
- **Analytics**: PostHog (`posthog-react-native`, EU host) - events, screens, error tracking, feature flags
- **Ratings**: `expo-store-review`
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
│   │   ├── notification-permission.tsx
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
│   │   └── ads-context.tsx
│   ├── hooks/
│   │   ├── use-notifications.ts
│   │   └── use-color-scheme.ts
│   ├── lib/
│   │   ├── notifications.ts
│   │   ├── purchases.ts        # expo-iap (StoreKit 2)
│   │   ├── revenuecat.ts       # observer mode
│   │   ├── premium-cache.ts
│   │   ├── ads.ts
│   │   ├── ads-consent.ts      # UMP + ATT
│   │   ├── analytics.ts        # PostHog
│   │   ├── rating.ts           # expo-store-review
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
├── store.config.json        # EAS Metadata (App Store listing)
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
- RTL: pass `unstable_nativeProps={{ direction: I18nManager.isRTL ? 'rtl' : 'ltr' }}`, the native bar does not follow `I18nManager` on its own
- If NativeTabs misbehaves on Android, fall back to expo-router `Tabs` **on Android only** (`if (Platform.OS === 'ios') return <NativeTabs>...`) with `IconSymbol` icons and a haptic tab button; iOS always uses NativeTabs

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

`eas.json` baseline:

```json
{
  "cli": { "version": ">= 16.28.0", "appVersionSource": "remote", "promptToConfigurePushNotifications": false },
  "build": {
    "development": { "developmentClient": true, "distribution": "internal" },
    "preview": { "distribution": "internal" },
    "production": { "autoIncrement": true }
  },
  "submit": { "production": { "ios": { "ascAppId": "<APP_STORE_CONNECT_APP_ID>" } } }
}
```

```bash
eas build --profile development --platform ios
eas build --profile development --platform android
eas build --profile production --platform ios
eas build --profile production --platform android
eas submit --platform ios
eas submit --platform android
eas metadata:push   # push store.config.json (title, subtitle, keywords, description per locale)
```

## Important Modules

### Purchases: StoreKit 2 via expo-iap + RevenueCat observer mode

- `src/lib/purchases.ts`: everything about buying and premium goes through `expo-iap` (`initConnection`, `fetchProducts`, `requestPurchase`, `finishTransaction`, `getActiveSubscriptions`, `getAvailablePurchases`, `restorePurchases`)
- `src/lib/revenuecat.ts`: RevenueCat is configured with `purchasesAreCompletedBy: { type: PURCHASES_ARE_COMPLETED_BY_TYPE.MY_APP, storeKitVersion: STOREKIT_VERSION.STOREKIT_2 }`. It only records purchases (`Purchases.recordPurchase(productId)`) and Apple Ads attribution (`enableAdServicesAttributionTokenCollection()`, no ATT needed). It never decides premium; if RevenueCat is down, purchases still work
- Products: `weekly` + `yearly` subscriptions (same subscription group) and an optional `lifetime` non-consumable
- Premium = an active subscription OR a lifetime purchase. Cache the last value (`premium-cache.ts`) so background tasks can read it without hitting the store

```ts
// Listen once at startup; renewals, interrupted purchases and Ask to Buy approvals arrive here too.
// Unfinished transactions are re-delivered on every launch.
purchaseUpdatedListener((purchase) => {
  if (purchase.purchaseState === 'pending') return notify(purchase); // Ask to Buy: wait for approval
  void recordRevenueCatPurchase(purchase.productId) // with a ~5s timeout, before finishing
    .then(() => finishTransaction({ purchase, isConsumable: false }))
    .finally(() => notify(purchase));
});

// Purchase
requestPurchase({
  request: { apple: { sku: productId }, google: { skus: [productId] } },
  type: productId === PRODUCT_IDS.LIFETIME ? 'in-app' : 'subs',
});
// Errors come from purchaseErrorListener; ErrorCode.UserCancelled => 'cancelled', not an error alert
```

- Result type: `'purchased' | 'pending' | 'cancelled' | 'failed'`; only `failed` shows an error alert
- Free trial: show it only if `isEligibleForIntroOfferIOS(subscriptionGroupId)` is true; derive days from `introductoryPricePaymentModeIOS === 'free-trial'` + period
- Add a `.storekit` configuration file to the Xcode scheme to test purchases on the simulator
- RevenueCat attribute `$posthogUserId` = PostHog distinct id, so purchases join analytics users

### AdMob

- Files: `src/lib/ads.ts` (unit IDs per platform), `src/lib/ads-consent.ts`, `src/context/ads-context.tsx`, `src/components/banner-ad.tsx`
- Ads disabled for premium users and in development (opt in with `EXPO_PUBLIC_ADS_DEV=1`)
- Test IDs must be used in development

### Onboarding & Paywall Flow (CRITICAL)

Flow: `Onboarding → Permission screens (location?, notifications) → Paywall → Main App (tabs)`

- Each step has a persisted flag in `OnboardingProvider` (`hasCompletedOnboarding`, `hasShownNotificationPermission`, `hasShownPaywall`, ... and `completeX()` setters)
- The root `_layout.tsx` is the single router guard: an effect watches the flags + `useSegments()` and `router.replace`s to the first unfinished step
- Until the current segment matches the expected step, render a loading view instead of the `Stack` (prevents a flash of the tabs)
- Permission screens explain the benefit first, then call the system prompt; "Not now" also completes the step
- The paywall knows it is in the onboarding flow via `!hasShownPaywall`: closing it calls `completePaywall()` + `router.replace('/')`. Opened later (settings, a locked feature) it just does `router.back()`. Pass `?from=settings|feature` for analytics

```tsx
useEffect(() => {
  if (isLoading) return;
  const step = segments[0];
  if (!hasCompletedOnboarding && step !== 'onboarding') router.replace('/onboarding');
  else if (hasCompletedOnboarding && !hasShownNotificationPermission && step !== 'notification-permission')
    router.replace('/notification-permission');
  else if (hasShownNotificationPermission && !hasShownPaywall && step !== 'paywall') router.replace('/paywall');
}, [isLoading, hasCompletedOnboarding, hasShownNotificationPermission, hasShownPaywall, segments]);
```

### Paywall (REQUIRED)

Plans, all with prices from the store (never hardcoded):

1. **Weekly**
2. **Yearly** - **selected by default** and highlighted; badge shows the real saving computed from store prices (`1 - yearly.price / (weekly.price * 52)`), and the free trial if eligible
3. **Lifetime** (optional non-consumable)

```tsx
const [selectedPlan, setSelectedPlan] = useState<'weekly' | 'yearly' | 'lifetime'>('yearly');
const [plans, setPlans] = useState<Plan[]>([]); // { id, displayPrice, price, currency, trialDays }

useEffect(() => {
  getPlans().then(setPlans); // open the page immediately, fill prices when they arrive
}, []);
```

App Store review (Guideline 3.1.2) requirements - the paywall MUST have:

- A visible close button (and onboarding must continue without buying)
- **Restore Purchases** button
- Price and period for every plan, trial length and what it renews into
- Auto-renewal text ("cancel anytime in App Store settings")
- Tappable **Privacy Policy** and **Terms of Use (EULA)** links (same links in the App Store description)
- After success: `refreshPremiumStatus()` from `useAds()`, then continue the flow / go back

### Settings Screen Options (REQUIRED)

Settings screen MUST include:

1. **Language** - Change app language
2. **Theme** - Light/Dark/System
3. **Notifications** - Enable/disable notifications
4. **Remove Ads** - Navigate to paywall (hidden if already premium)
5. **Restore Purchases**
6. **Privacy choices** - `showPrivacyOptions()` (only when `privacyOptionsRequired`, EU)
7. **Analytics** toggle - PostHog opt-in/opt-out (only when analytics is configured)
8. **Rate / Share / Privacy Policy / Terms / Support** links
9. **Reset Onboarding** - Restart onboarding flow (`__DEV__` only)

```tsx
const { isPremium } = useAds();

// Remove Ads - navigates to paywall
const handleRemoveAds = () => {
  router.push('/paywall');
};

// Reset onboarding
const handleResetOnboarding = async () => {
  await setOnboardingCompleted(false);
  router.replace('/onboarding');
};

// In settings list:
{!isPremium && (
  <SettingsItem
    title={t('settings.removeAds')}
    icon="crown.fill"
    onPress={handleRemoveAds}
  />
)}

<SettingsItem
  title={t('settings.resetOnboarding')}
  icon="arrow.counterclockwise"
  onPress={handleResetOnboarding}
/>
```

## Localization

- File: `src/lib/i18n.ts`, strings in `src/locales/*.json`
- Initial language: walk `Localization.getLocales()` in order and take the first supported one (a user's second language may be supported), fallback `en`
- Persist the user's choice and load it on startup
- Translate iOS permission texts / app name with `expo.locales` in `app.json` (`{ "tr": "./locales/ios/tr.json" }`) + `CFBundleAllowMixedLocalizations: true`

### RTL (Arabic, Hebrew, Persian, Urdu)

Switch direction at runtime when the language changes:

```ts
I18nManager.allowRTL(rtl);
I18nManager.forceRTL(rtl);
if (Platform.OS === 'ios') Settings.set({ AppleLanguages: [lang] }); // native strings, share sheet, alerts
if (I18nManager.isRTL !== rtl) {
  // Guard with a stored flag so a failed reload cannot loop
  if (__DEV__) DevSettings.reload();
  else await Updates.reloadAsync();
}
```

- Use `start`/`end` (`marginStart`, `paddingEnd`) instead of `left`/`right`
- Flip directional icons (chevrons, arrows) with `transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }]`

## Analytics (PostHog)

File: `src/lib/analytics.ts`

```ts
const posthog = process.env.EXPO_PUBLIC_POSTHOG_KEY
  ? new PostHog(process.env.EXPO_PUBLIC_POSTHOG_KEY, {
      host: process.env.EXPO_PUBLIC_POSTHOG_HOST ?? 'https://eu.i.posthog.com',
      disabled: __DEV__ && process.env.EXPO_PUBLIC_POSTHOG_DEV !== '1',
      captureAppLifecycleEvents: true,
      errorTracking: { autocapture: { uncaughtExceptions: true, unhandledRejections: true, nativeCrashes: true } },
    })
  : null;

type Events = {
  onboarding_completed: { skipped: boolean };
  paywall_viewed: { from: 'onboarding' | 'settings' | 'feature' };
  purchase_started: { plan: 'weekly' | 'yearly' | 'lifetime' };
  purchase_completed: { plan: 'weekly' | 'yearly' | 'lifetime'; revenue?: number; currency?: string; trial?: boolean };
  purchase_failed: { plan: 'weekly' | 'yearly' | 'lifetime'; reason: 'pending' | 'cancelled' | 'failed' };
  restore_completed: { active: boolean };
  // ...one entry per product event
};

export function track<E extends keyof Events>(event: E, properties: Events[E]) {
  posthog?.capture(event, properties);
}
export const trackScreen = (name: string) => void posthog?.screen(name);
```

- Typed event map - no free-form event names
- Track screens from the root layout: `useEffect(() => trackScreen(pathname), [pathname])`
- Track notification opens with `Notifications.useLastNotificationResponse()` (dedupe by id + date)
- Anonymous only: no PII, no exact coordinates, no user-typed text
- Feature flags (`posthog.getFeatureFlag`) for remote config / A-B tests; always have a default
- Keys live in `.env` (`EXPO_PUBLIC_POSTHOG_KEY`, `EXPO_PUBLIC_POSTHOG_HOST`), never committed

## Ratings (expo-store-review)

File: `src/lib/rating.ts`

- Ask at a **value moment** (task completed, goal reached), never on launch or after an error
- Conditions: app opened on at least 3 different days, 60+ days since the last request
- Delay ~1.2s so the success animation finishes, check `StoreReview.isAvailableAsync()`, then `StoreReview.requestReview()`
- iOS shows it at most 3 times a year and does not report whether it was shown
- Settings "Rate us" opens `https://apps.apple.com/app/id<ID>?action=write-review` instead

## Notifications

- Files: `src/lib/notifications.ts`, `src/hooks/use-notifications.ts`
- Remote push needs the `aps-environment` entitlement; local notifications do not
- iOS keeps at most 64 pending local notifications: schedule a rolling window and re-sync on every `AppState` → `active` and from the background task
- Custom sounds: list `.caf` (iOS) / `.wav` files under the `expo-notifications` plugin `sounds` option, refer to them by file name; keep them under 30 s
- Ask permission on the pre-permission screen, not on launch

## Background Tasks

```ts
// src/lib/background-sync.ts - imported from app/_layout.tsx so defineTask runs at module load
TaskManager.defineTask(TASK_NAME, async () => {
  try {
    await syncScheduledContent();
    return BackgroundTask.BackgroundTaskResult.Success;
  } catch {
    return BackgroundTask.BackgroundTaskResult.Failed;
  }
});
await BackgroundTask.registerTaskAsync(TASK_NAME, { minimumInterval: 60 }); // minutes, a lower bound only
```

- Packages: `expo-background-task` + `expo-task-manager`
- iOS decides when it runs (no guarantee); the real refresh happens on foreground, the task is a safety net
- Never touch UI-only APIs (RTL reload, prompts) from a task

## Widgets & Live Activities (iOS, optional)

- `expo-widgets` + `@expo/ui/swift-ui`: widgets are JSX functions with a `'widget'` directive; they run in the extension, so no hooks, imports or outer constants inside - everything comes from props
- Configure in `app.json` under the `expo-widgets` plugin (`groupIdentifier`, widget `name`, `supportedFamilies`) and use the same App Group in code
- Push data with `Widget.updateTimeline(entries)` (precompute several days of entries so the widget keeps working when the app is not opened)
- Extensions can only read files from the App Group container: copy images there (`Paths.appleSharedContainers[APP_GROUP]`), version the file name when the image changes
- Live Activities (iOS 16.2+) can only be **started** while the app is in the foreground; background tasks may only update a running one

## Coding Standards

- Use functional components
- Strict TypeScript
- Avoid hardcoded strings
- File names kebab-case (`use-prayer-times.ts`), components PascalCase
- Unit tests for pure logic with `bun test` (`"test": "bun test"`, `@types/bun`)
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

## useColorScheme Hook

File: `src/hooks/use-color-scheme.ts`

```tsx
import { useThemeContext } from '@/context/theme-context';

export function useColorScheme(): 'light' | 'dark' | 'unspecified' {
  const { isDark } = useThemeContext();
  return isDark ? 'dark' : 'light';
}
```

## Important Notes

1. iOS permissions are defined in `app.json`
2. Android permissions are defined in `app.json`
3. New Architecture is the only architecture since SDK 55 - no `newArchEnabled` flag needed
4. Enable typed routes and the React Compiler via `experiments.typedRoutes` / `experiments.reactCompiler`
5. Secrets and keys go to `.env` as `EXPO_PUBLIC_*` (and EAS environment variables for builds)

## App Store & Play Store Notes

- iOS ATT permission required before personalised ads (after the UMP form)
- Restore purchases must work correctly
- Paywall must meet Guideline 3.1.2 (see Paywall section); App Store description must also include the auto-renew text, Privacy Policy and Terms (EULA) links
- Do not request permissions the app does not use (5.1.1); every permission needs a purpose string
- `ITSAppUsesNonExemptEncryption: false` to skip the export compliance question
- Store listing lives in `store.config.json` (EAS Metadata), localized per language
- Target SDK must be up to date

## Store Screenshots

When the user asks for App Store / Google Play screenshots or store visuals, ALWAYS use the `/app-store-screenshot` skill (invoke it with the Skill tool) instead of building the images by hand.

1. Take raw screenshots from the simulator (`xcrun simctl io booted screenshot <file>.png`), one per key screen, in every store language
2. Run `/app-store-screenshot` with those screenshots; it adds the device frame, background and headline while keeping the app UI pixel-exact
3. Keep the output in `store/screenshots/` next to `store.config.json`

## Testing Checklist

- UI tested in all languages
- Dark / Light mode
- Notifications
- Premium flow (StoreKit config file on simulator, Sandbox account on device)
- Free trial eligibility, Ask to Buy (pending), cancelled purchase
- Restore purchases
- EU consent form (`EXPO_PUBLIC_ADS_DEBUG_EEA=1`) and ATT prompt
- RTL layout
- Offline support
- Multiple screen sizes

## After Development

```bash
npx expo prebuild --clean
bun ios
bun android
```

> NOTE: `prebuild --clean` recreates ios and android folders. Run it after modifying native modules or app.json.
