# React Native: Intercom to Gleap Migration

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
npm uninstall @intercom/intercom-react-native
```

### iOS
```
cd ios && pod install
```

### Android
- Remove `IntercomModule.initialize(this, "apiKey", "appId")` from `MainApplication.java`
- Remove `MainNotificationService` and its manifest entry if only used by Intercom
- Remove `com.google.firebase:firebase-messaging` dependency if only used by Intercom

### Clean up
- Remove all `import Intercom from '@intercom/intercom-react-native'`
- Remove Intercom push notification handling code

## Install Gleap

```
npm install react-native-gleapsdk --save
```

### iOS
```
cd ios && pod install
```

### Android
Open Android Studio and sync Gradle.

## Initialization

Intercom's native init + JS `loginUser`. Gleap uses JS-only init + identify.

**Before (Intercom):**
```java
// MainApplication.java (Android)
IntercomModule.initialize(this, "apiKey", "appId");
```
```objc
// AppDelegate.m (iOS)
[IntercomModule initialize:@"apiKey" withAppId:@"appId"];
```
```javascript
// JavaScript
import Intercom from '@intercom/intercom-react-native';
Intercom.loginUserWithUserAttributes({ email: "user@example.com", userId: "123" });
```

**After (Gleap):**
```javascript
// index.js (JavaScript only, no native init needed)
import Gleap from 'react-native-gleapsdk';

Gleap.initialize("GLEAP_API_KEY");

// After login
Gleap.identify("123", {
  email: "user@example.com",
  name: "John Doe",
});
```

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom.loginUserWithUserAttributes({ userId, email })` | `Gleap.identify(userId, { email, ... })` |
| `Intercom.loginUnidentifiedUser()` | Not needed (Gleap uses guest sessions) |
| `Intercom.logout()` | `Gleap.clearIdentity()` |
| `Intercom.updateUser({ name, email, ... })` | `Gleap.updateContact({ name, email, ... })` |
| `Intercom.updateUser({ customAttributes: {} })` | `Gleap.updateContact({ customData: {} })` |
| `Intercom.updateUser({ companies: [{}] })` | `Gleap.updateContact({ companyId, companyName })` |

### Attribute mapping

| Intercom | Gleap |
|----------|-------|
| `userId` | First arg to `identify()` |
| `email` | `email` |
| `name` | `name` |
| `phone` | `phone` |
| `signedUpAt` (unix) | `createdAt` (Date) |
| `languageOverride` | Use `Gleap.setLanguage()` separately |
| `companies[0].id` | `companyId` |
| `companies[0].name` | `companyName` |
| `companies[0].plan` | `plan` |
| `companies[0].monthlySpend` | `value` |
| `customAttributes` | `customData` |

## Event Tracking

| Intercom | Gleap |
|----------|-------|
| `Intercom.logEvent('name')` | `Gleap.trackEvent('name')` |
| `Intercom.logEvent('name', { key: 'val' })` | `Gleap.trackEvent('name', { key: 'val' })` |

## Custom Data

| Intercom | Gleap |
|----------|-------|
| `Intercom.updateUser({ customAttributes: {} })` | `Gleap.attachCustomData({})` |
| No separate API | `Gleap.setCustomData('key', 'value')` |
| No separate API | `Gleap.removeCustomData('key')` |

## Widget Control

| Intercom | Gleap |
|----------|-------|
| `Intercom.displayMessenger()` | `Gleap.open()` |
| `Intercom.displayHelpCenter()` | `Gleap.openHelpCenter()` |
| `Intercom.displayArticle(id)` | `Gleap.openHelpCenterArticle(id)` |
| `Intercom.setInAppMessageVisibility('GONE')` | `Gleap.showFeedbackButton(false)` |

## Permissions

Both iOS and Android permissions are similar between Intercom and Gleap. Verify that `Info.plist` and `AndroidManifest.xml` still have the photo/camera/microphone permissions needed by Gleap.
