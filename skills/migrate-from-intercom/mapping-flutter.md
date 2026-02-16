# Flutter: Intercom to Gleap Migration

## Table of Contents
- [Remove Intercom](#remove-intercom)
- [Install Gleap](#install-gleap)
- [Initialization](#initialization)
- [User Identity](#user-identity)
- [Event Tracking](#event-tracking)
- [Custom Data](#custom-data)
- [Widget Control](#widget-control)

## Remove Intercom

```
flutter pub remove intercom_flutter
```

### iOS
```
cd ios && pod install
```

### Android
- Remove `Intercom.initialize(this, ...)` from custom Application class if present
- Remove `FlutterFragmentActivity` requirement if only needed for Intercom
- Remove `android.useAndroidX=true` / `android.enableJetifier=true` from `gradle.properties` only if no other dependency needs them

### Clean up
- Remove `import 'package:intercom_flutter/intercom_flutter.dart'`
- Remove all `Intercom.instance.*` calls

## Install Gleap

```
flutter pub add gleap_sdk
```

### iOS
```
cd ios && pod install
```

### Web (if applicable)
Add Gleap CDN script to `web/index.html` `<head>`:
```html
<script>
!function(){if(!(window.Gleap=window.Gleap||[]).invoked){window.GleapActions=[];var e=new Proxy({invoked:!0},{get:function(e,n){return"invoked"===n?e.invoked:function(){var e=Array.prototype.slice.call(arguments);window.GleapActions.push({e:n,a:e})}},set:function(e,n,t){return e[n]=t,!0}});window.Gleap=e;var n=document.getElementsByTagName("head")[0],t=document.createElement("script");t.type="text/javascript",t.async=!0,t.src="https://sdk.gleap.io/latest/index.js",n.appendChild(t)}}();
</script>
```

Then: `flutter clean && flutter pub get`

## Initialization

**Before (Intercom):**
```dart
import 'package:intercom_flutter/intercom_flutter.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Intercom.instance.initialize('appId',
    iosApiKey: 'iosKey', androidApiKey: 'androidKey');
  runApp(App());
}
```

**After (Gleap):**
```dart
import 'package:gleap_sdk/gleap_sdk.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  Gleap.initialize(token: 'GLEAP_API_KEY');
  runApp(App());
}
```

Note: Intercom needs separate iOS/Android keys. Gleap uses a single API key.

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom.instance.loginIdentifiedUser(userId:, email:)` | `Gleap.identify(userId:, userProperties: GleapUserProperty(...))` |
| `Intercom.instance.loginUnidentifiedUser()` | Not needed (Gleap uses guest sessions) |
| `Intercom.instance.logout()` | `Gleap.clearIdentity()` |
| `Intercom.instance.updateUser(...)` | `Gleap.updateContact(userProperties: GleapUserProperty(...))` |
| `Intercom.instance.setUserHash(hash)` | Pass `userHash:` arg to `Gleap.identify()` |

**Before (Intercom):**
```dart
await Intercom.instance.loginIdentifiedUser(userId: '123', email: 'user@example.com');
```

**After (Gleap):**
```dart
Gleap.identify(
  userId: '123',
  userProperties: GleapUserProperty(
    email: 'user@example.com',
    name: 'John Doe',
    plan: 'Pro',
    companyId: 'c1',
    companyName: 'ACME',
    customData: <String, dynamic>{'role': 'admin'},
  ),
);
```

## Event Tracking

| Intercom | Gleap |
|----------|-------|
| `Intercom.instance.logEvent('name')` | `Gleap.trackEvent(name: 'name')` |
| No metadata support in Flutter SDK | `Gleap.trackEvent(name: 'name', data: {...})` |

Gleap's Flutter SDK supports event metadata, which Intercom's Flutter SDK does not.

## Custom Data

| Intercom | Gleap |
|----------|-------|
| Custom attributes in `updateUser` | `customData` in `GleapUserProperty` |
| No separate API | `Gleap.attachCustomData(customData: {...})` |
| No separate API | `Gleap.setCustomData(key: 'k', value: 'v')` |
| No separate API | `Gleap.removeCustomDataForKey(key: 'k')` |

## Widget Control

| Intercom | Gleap |
|----------|-------|
| `Intercom.instance.displayMessenger()` | `Gleap.open()` |
| `Intercom.instance.displayHelpCenter()` | `Gleap.openHelpCenter()` |
| `Intercom.instance.displayArticle(id)` | `Gleap.openHelpCenterArticle(id)` |
| `Intercom.instance.displayMessages()` | `Gleap.openConversations()` |
| `Intercom.instance.displayHome()` | `Gleap.open()` |
| `Intercom.instance.hideMessenger()` | `Gleap.close()` |
| `Intercom.instance.setLauncherVisibility(visible)` | `Gleap.showFeedbackButton(visible)` |
| `Intercom.instance.unreadConversationCount()` | Subscribe via callbacks |

## Permissions

Same as Intercom. Verify these are still in `Info.plist` and `AndroidManifest.xml`:
- Photo library / camera / microphone (iOS)
- `READ_MEDIA_IMAGES` / `READ_EXTERNAL_STORAGE` (Android)
