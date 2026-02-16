---
name: migrate-from-intercom
description: Migrates projects from Intercom to Gleap. Detects Intercom SDK integration (JavaScript, iOS, Android, React Native, Flutter), removes it, installs the Gleap SDK, and replaces all Intercom API calls with Gleap equivalents. Also covers server-side REST API migration. Use when switching from Intercom to Gleap, replacing Intercom, or migrating away from Intercom.
---

# Migrate from Intercom to Gleap

Guides users through replacing Intercom with Gleap in their codebase. Covers client-side SDK swap and server-side API migration.

## Detect Intercom

Search the codebase for Intercom integration. Use `Grep` to find:

### Package references
- `package.json`: `@intercom/messenger-js-sdk`, `intercom-client`, `@intercom/intercom-react-native`
- `Podfile`: `pod 'Intercom'`
- `build.gradle`: `io.intercom.android:intercom-sdk`
- `pubspec.yaml`: `intercom_flutter`
- `composer.json`: `intercom/intercom-php`
- `Gemfile`: `intercom-rails`, `intercom-ruby`

### Code patterns
- JavaScript: `Intercom(`, `window.intercomSettings`, `widget.intercom.io`, `@intercom/messenger-js-sdk`
- iOS: `import Intercom`, `Intercom.setApiKey`, `Intercom.loginUser`
- Android: `import com.intercom`, `Intercom.initialize`, `Intercom.client()`
- React Native: `import Intercom from '@intercom/intercom-react-native'`, `IntercomModule`
- Flutter: `import 'package:intercom_flutter'`, `Intercom.instance`
- Server: `intercom-client`, `intercom-php`, `intercom-ruby`, `intercom-go`, `intercom-java`, `Intercom.Dotnet`

Report all findings to the user before proceeding.

## Workflow

1. **Detect Intercom** using the patterns above. Report which platforms and files are affected.
2. **Confirm scope** with the user: which parts to migrate (client SDK, server API, or both).
3. **Read the mapping file** for the detected platform from this skill's directory.
4. **Remove Intercom SDK**: Uninstall packages, remove script tags, delete imports and configuration.
5. **Install Gleap SDK**: Follow the installation steps from the mapping file (or reference the `gleap-sdk-setup` skill if installed).
6. **Replace API calls**: Use the mapping tables to convert every Intercom call to its Gleap equivalent.
7. **Migrate server-side code** if applicable: Replace Intercom REST API calls with Gleap REST API calls. See `mapping-server-api.md`.
8. **Clean up**: Remove Intercom-specific config (API keys, app IDs, `intercomSettings`, push notification handlers).
9. **Verify**: Build and run the project. Search for any remaining `intercom` references.

## Mapping Files

Read the appropriate file based on detected platform:

- **JavaScript / Web**: `mapping-javascript.md`
- **iOS (Swift / Objective-C)**: `mapping-ios.md`
- **Android (Kotlin / Java)**: `mapping-android.md`
- **React Native**: `mapping-react-native.md`
- **Flutter**: `mapping-flutter.md`
- **Server-side REST API**: `mapping-server-api.md`

For projects using multiple platforms (e.g., React Native with server API), read all relevant files.

## Important Notes

- Intercom uses `app_id` + `api_key` (separate for iOS/Android). Gleap uses a single API key per project from https://app.gleap.io
- Intercom's `boot()` combines initialization + user identification. In Gleap, these are separate: `initialize()` then `identify()`
- Intercom's `shutdown()` = Gleap's `clearIdentity()`
- Intercom's `update()` = Gleap's `updateContact()`
- Intercom custom user attributes map to Gleap's `customData` object (max 35 keys, primitives only)
- Intercom company data maps to Gleap's `companyId` + `companyName` fields in identify
- Features without direct Gleap equivalent: product tours (`startTour`), tickets view (`showTicket`), news items (`showNews`). Note these to the user as requiring alternative approaches
- Server-side: Intercom REST API uses `Bearer` token auth. Gleap uses `Api-Token` header (legacy) or `Bearer` + `Project` headers (v3 API)
