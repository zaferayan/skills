# Expo Mobile App Skill

A Claude Code skill for building production-ready Expo React Native mobile applications.

## Features

- **StoreKit 2 (expo-iap)** - Weekly/yearly subscriptions + lifetime purchase, trial eligibility, Ask to Buy
- **RevenueCat (observer mode)** - Purchase records and Apple Ads attribution
- **AdMob** - Banner ads with UMP (GDPR) consent, ATT and premium user detection
- **PostHog** - Typed analytics events, screen tracking, error tracking, feature flags
- **i18n** - Multi-language support with RTL (Turkish/English/Arabic)
- **Onboarding** - Fullscreen video background, permission screens, then paywall
- **Paywall** - Store prices, yearly default, App Store 3.1.2 compliant
- **Ratings** - Native review prompt at value moments
- **Background tasks, widgets & Live Activities** (optional)
- **NativeTabs** - Native tab navigation
- **Theme** - Light/Dark/System mode support

## Installation

```bash
npx skills add zaferayan/skills
```

## What it does

When you ask Claude Code to create an Expo app, this skill ensures:

1. **Required Screens**: Onboarding, Paywall, Settings
2. **Proper Flow**: Onboarding → Paywall → Main App
3. **Monetization**: StoreKit 2 (expo-iap) + RevenueCat observer + AdMob with consent
4. **Best Practices**: No AsyncStorage, no lineHeight, NativeTabs only

## Usage

After installing, simply ask Claude Code:

```
/zafer-skills Create a water reminder app
```

Claude will automatically:
- Set up the project structure
- Configure StoreKit purchases, RevenueCat observer mode, AdMob consent and PostHog
- Create onboarding with video background
- Build paywall with subscription options
- Add settings with language, theme, and reset options

## Tech Stack

- Expo SDK 57 / React Native 0.86
- Expo Router (file-based routing)
- NativeTabs navigation
- expo-iap (StoreKit 2 / Play Billing)
- RevenueCat (react-native-purchases, observer mode)
- AdMob (react-native-google-mobile-ads) + expo-tracking-transparency
- PostHog (posthog-react-native)
- expo-store-review
- i18next + react-i18next
- expo-video
- expo-sqlite (localStorage polyfill)

## License

MIT
