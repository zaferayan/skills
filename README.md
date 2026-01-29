# Expo Mobile App Skill

A Claude Code skill for building production-ready Expo React Native mobile applications.

## Features

- **RevenueCat** - In-app purchases with weekly/yearly subscriptions
- **AdMob** - Banner ads with premium user detection
- **i18n** - Multi-language support (Turkish/English)
- **Onboarding** - Fullscreen video background with swipe screens
- **Paywall** - Subscription options with 50% OFF yearly badge
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
3. **Monetization**: RevenueCat + AdMob integration
4. **Best Practices**: No AsyncStorage, no lineHeight, NativeTabs only

## Usage

After installing, simply ask Claude Code:

```
/zafer-skills Create a water reminder app
```

Claude will automatically:
- Set up the project structure
- Configure RevenueCat and AdMob
- Create onboarding with video background
- Build paywall with subscription options
- Add settings with language, theme, and reset options

## Tech Stack

- Expo SDK
- Expo Router (file-based routing)
- NativeTabs navigation
- RevenueCat (react-native-purchases)
- AdMob (react-native-google-mobile-ads)
- i18next + react-i18next
- expo-video
- expo-sqlite (localStorage polyfill)

## License

MIT
