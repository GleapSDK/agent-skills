# Flutter SDK

## Table of Contents
- [Installation](#installation)
- [Platform Setup](#platform-setup)
- [Initialization](#initialization)
- [Required Permissions](#required-permissions)
- [Common API Usage](#common-api-usage)

## Installation

```
flutter pub add gleap_sdk
```

### Flutter v2 (< v3)

Add to `pubspec.yaml`:

```yaml
dependencies:
  gleap_sdk:
    git:
      url: https://github.com/GleapSDK/Flutter-SDK.git
      ref: flutter-v2
```

## Platform Setup

### iOS

```
cd ios && pod install
```

### Android

If version conflict occurs, add to `AndroidManifest.xml`:

```xml
<manifest ... xmlns:tools="http://schemas.android.com/tools">
  <uses-sdk android:minSdkVersion="21"
    tools:overrideLibrary="io.gleap.gleap_sdk"/>
  <application ... tools:overrideLibrary="io.gleap.gleap_sdk">
```

Do NOT set `android:hardwareAccelerated="false"` at the application level.

### Web

Add as the first element in the `<head>` of `web/index.html`:

```html
<script>
!function(){if(!(window.Gleap=window.Gleap||[]).invoked){window.GleapActions=[];var e=new Proxy({invoked:!0},{get:function(e,n){return"invoked"===n?e.invoked:function(){var e=Array.prototype.slice.call(arguments);window.GleapActions.push({e:n,a:e})}},set:function(e,n,t){return e[n]=t,!0}});window.Gleap=e;var n=document.getElementsByTagName("head")[0],t=document.createElement("script");t.type="text/javascript",t.async=!0,t.src="https://sdk.gleap.io/latest/index.js",n.appendChild(t)}}();
</script>
```

Then run `flutter clean && flutter pub get`.

## Initialization

In `main.dart`, ensure the binding is initialized first:

```dart
import 'package:gleap_sdk/gleap_sdk.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  Gleap.initialize(projectId: 'PROJECT_ID');
  runApp(MyApp());
}
```

Replace `PROJECT_ID` with the Project ID (`projectId:` needs gleap_sdk 19.5.0 or newer; older versions take the same value as `token:`).

## Required Permissions

### Android (`AndroidManifest.xml`)

Image attachments:
```xml
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
```

Audio recordings:
```xml
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-feature android:name="android.hardware.audio.pro" android:required="false" />
<uses-feature android:name="android.hardware.microphone" android:required="false" />
```

### iOS (`Info.plist`)

```xml
<key>NSPhotoLibraryUsageDescription</key>
<string>Allow photo access for attachments</string>

<key>NSCameraUsageDescription</key>
<string>Allow camera access for screenshots</string>

<key>NSMicrophoneUsageDescription</key>
<string>Allow microphone access for audio feedback</string>
```

## Common API Usage

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start:

```dart
// after login and on every app start while signed in; `token` comes from your backend
await Gleap.setUserJwt(token: token);
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout. On an installed SDK without `setUserJwt`, pass the token where the legacy user hash goes:

```dart
Gleap.identify(userId: 'USER_ID', userProperties: GleapUserProperty(), userHash: token);
```

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```dart
Gleap.identify(
  userId: 'USER_ID',
  userProperties: GleapUserProperty(
    name: 'Franz',
    email: 'franz@example.com',
    customData: <String, dynamic>{
      'role': 'admin',
    },
  ),
);
```

Legacy user hash (existing integrations; a server-side HMAC-SHA256 of the user id with the project secret, accepted on existing projects while "Accept legacy user hash" is on; projects created in Signed mode don't accept it):
```dart
Gleap.identify(
  userId: 'USER_ID',
  userProperties: GleapUserProperty(name: 'Franz', email: 'franz@example.com'),
  userHash: 'GENERATED_USER_HASH',
);
```

Update contact (partial updates, unsigned; fields marked "Only signed updates" stay unchanged):
```dart
Gleap.updateContact(
  userProperties: GleapUserProperty(customData: <String, dynamic>{'onboardingStep': 3}),
);
```

Clear identity on logout:
```dart
Gleap.clearIdentity();
```

### Track Events

```dart
Gleap.trackEvent(name: 'User signed in');

// With data:
Gleap.trackEvent(
  name: 'Purchase completed',
  data: <String, dynamic>{'amount': 49.99, 'currency': 'USD'},
);
```

### Custom Data

```dart
Gleap.attachCustomData(customData: <String, dynamic>{'environment': 'staging'});

// Incremental:
Gleap.setCustomData(key: 'key', value: 'value');

// Remove:
Gleap.removeCustomDataForKey(key: 'key');

// Clear all:
Gleap.clearCustomData();
```

### Widget Control

```dart
Gleap.open();
Gleap.close();
Gleap.isOpened();
```
