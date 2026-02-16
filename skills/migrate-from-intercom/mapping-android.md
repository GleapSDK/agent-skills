# Android: Intercom to Gleap Migration

## Table of Contents
- [Remove Intercom](#remove-intercom)
- [Install Gleap](#install-gleap)
- [Initialization](#initialization)
- [User Identity](#user-identity)
- [Event Tracking](#event-tracking)
- [Custom Data](#custom-data)
- [Widget Control](#widget-control)

## Remove Intercom

Remove from `build.gradle` (Module: app):
```gradle
// DELETE:
implementation 'io.intercom.android:intercom-sdk:X.X.X'
// or
implementation 'io.intercom.android:intercom-sdk-base:X.X.X'
```

Sync Gradle.

### Clean up
- Remove `import com.intercom.android.sdk.Intercom` from all files
- Remove `Intercom.initialize(this, ...)` from Application class
- Remove Intercom push notification service from `AndroidManifest.xml`
- Remove `<uses-permission android:name="android.permission.VIBRATE"/>` if only used by Intercom

## Install Gleap

Add to `build.gradle` (Module: app):
```gradle
implementation group: 'io.gleap', name: 'gleap-android-sdk', version: 'LATEST_VERSION'
```

Sync Gradle. Use `get-latest-versions.sh` from the `gleap-sdk-setup` skill for the current version.

## Initialization

Intercom initializes in Application class, then logs in separately. Gleap initializes in Activity.

**Before (Intercom):**
```kotlin
// Application class
Intercom.initialize(this, "API_KEY", "APP_ID")

// After login
val reg = Registration.create().withUserId("123").withEmail("user@example.com")
Intercom.client().loginIdentifiedUser(userRegistration = reg)
```

**After (Gleap):**
```java
import io.gleap.Gleap;

// MainActivity onCreate
Gleap.initialize("GLEAP_API_KEY", this);

// After login
GleapSessionProperties props = new GleapSessionProperties();
props.setEmail("user@example.com");
Gleap.getInstance().identifyContact("123", props);
```

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom.initialize(ctx, key, appId)` | `Gleap.initialize(key, activity)` |
| `Intercom.client().loginIdentifiedUser(reg)` | `Gleap.getInstance().identifyContact(userId, props)` |
| `Intercom.client().loginUnidentifiedUser()` | Not needed (Gleap uses guest sessions) |
| `Intercom.client().logout()` | `Gleap.getInstance().clearIdentity()` |
| `Intercom.client().updateUser(attrs)` | `Gleap.getInstance().updateContact(props)` |

### Attribute mapping

| Intercom (`Registration` / `UserAttributes`) | Gleap (`GleapSessionProperties`) |
|---------------------------------------------|----------------------------------|
| `.withUserId("id")` | First arg to `identifyContact` |
| `.withEmail("email")` | `.setEmail("email")` |
| `.withName("name")` | `.setName("name")` |
| `.withPhone("phone")` | `.setPhone("phone")` |
| `.withCompany(company)` | `.setCompanyId("id")`, `.setCompanyName("name")` |
| `.withCustomAttribute(key, val)` | `.setCustomData(jsonObject)` |

## Event Tracking

| Intercom | Gleap |
|----------|-------|
| `Intercom.client().logEvent("name")` | `Gleap.getInstance().trackEvent("name")` |
| `Intercom.client().logEvent("name", metadata)` | `Gleap.getInstance().trackEvent("name", jsonData)` |

Note: Intercom accepts a `Map<String, Object>`, Gleap accepts a `JSONObject`.

## Custom Data

| Intercom | Gleap |
|----------|-------|
| Custom attributes in `UserAttributes` | `props.setCustomData(jsonObject)` in identify |
| No separate custom data API | `Gleap.getInstance().attachCustomData(jsonObject)` |
| No separate custom data API | `Gleap.getInstance().setCustomData("val", "key")` |

## Widget Control

| Intercom | Gleap |
|----------|-------|
| `Intercom.client().displayMessenger()` | `Gleap.getInstance().open()` |
| `Intercom.client().displayHelpCenter()` | `Gleap.getInstance().openHelpCenter()` |
| `Intercom.client().displayArticle(id)` | `Gleap.getInstance().openHelpCenterArticle(id)` |
| `Intercom.client().setLauncherVisibility(GONE)` | `Gleap.getInstance().showFeedbackButton(false)` |

## AndroidManifest.xml

Add Gleap permissions (if not already present from Intercom):
```xml
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
```
