# Ionic / Capacitor SDK

## Table of Contents
- [Installation](#installation)
- [Initialization](#initialization)
- [Required Permissions](#required-permissions)
- [Common API Usage](#common-api-usage)

## Installation

Install based on your Capacitor version:

### Capacitor v5+

```
npm install capacitor-gleap-plugin
```

### Capacitor v4

```
npm install GleapSDK/Capacitor-SDK#capacitor-v4 --save
```

### Capacitor v3

```
npm install capacitor-gleap-plugin@8.2.3
```

### Sync (all versions)

```
npx cap sync
```

## Initialization

In your `index.js`, `index.tsx`, or main component (call only once):

```typescript
import { Gleap } from 'capacitor-gleap-plugin';

Gleap.initialize({ token: "API_KEY" });
```

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

The Ionic/Capacitor SDK uses the same JavaScript-style API as the web SDK.

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start:

```typescript
import { Gleap } from 'capacitor-gleap-plugin';

// after login and on every app start while signed in; `token` comes from your backend
await Gleap.setUserJwt({ token });
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout. On an installed SDK without `setUserJwt`, pass the token where the legacy user hash goes:

```typescript
Gleap.identify({ userId: "USER_ID", userHash: token });
```

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```typescript
Gleap.identify({
  userId: "USER_ID",
  userProperties: {
    name: "Franz",
    email: "franz@example.com",
    customData: { role: "admin" },
  },
});
```

Update contact (unsigned; fields marked "Only signed updates" stay unchanged):
```typescript
Gleap.updateContact({ userProperties: { customData: { onboardingStep: 3 } } });
```

Clear identity on logout:
```typescript
Gleap.clearIdentity();
```

### Track Events

```typescript
Gleap.trackEvent({ name: "User signed in" });

// With data:
Gleap.trackEvent({ name: "Purchase completed", data: { amount: 49.99 } });
```

### Custom Data

```typescript
Gleap.attachCustomData({ data: { environment: "staging" } });
Gleap.setCustomData({ key: "key", value: "value" });
Gleap.removeCustomData({ key: "key" });
Gleap.clearCustomData();
```

### Widget Control

```typescript
Gleap.open();
Gleap.close();
const { isOpened } = await Gleap.isOpened();
```
