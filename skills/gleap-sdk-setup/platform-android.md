# Android SDK

## Table of Contents
- [Installation](#installation)
- [Initialization](#initialization)
- [Required Permissions](#required-permissions)
- [Common API Usage](#common-api-usage)

## Installation

Add to `build.gradle` (Module: app) dependencies:

```gradle
implementation group: 'io.gleap', name: 'gleap-android-sdk', version: 'LATEST_ANDROID_VERSION'
```

Replace `LATEST_ANDROID_VERSION` with the version from `get-latest-versions.sh`.

Sync Gradle after adding the dependency.

## Initialization

In your main Activity's `onCreate`:

```java
import io.gleap.Gleap;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    Gleap.initialize("API_KEY", this);
}
```

## Required Permissions

Add to `AndroidManifest.xml`:

### Image attachments

```xml
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
```

### Audio recordings

```xml
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-feature android:name="android.hardware.audio.pro" android:required="false" />
<uses-feature android:name="android.hardware.microphone" android:required="false" />
```

### Video calls

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-feature android:name="android.hardware.audio.pro" android:required="false" />
<uses-feature android:name="android.hardware.microphone" android:required="false" />
<uses-feature android:name="android.hardware.camera" android:required="false" />
```

## Common API Usage

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start:

```java
// after login and on every app start while signed in; `token` comes from your backend
Gleap.getInstance().setUserJwt(token);
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout. On an installed SDK without `setUserJwt`, pass the token where the legacy user hash goes:

```java
GleapSessionProperties props = new GleapSessionProperties();
props.setHash(token);
Gleap.getInstance().identifyContact("USER_ID", props);
```

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```java
GleapSessionProperties sessionProperties = new GleapSessionProperties();
sessionProperties.setName("Franz");
sessionProperties.setEmail("franz@example.com");

JSONObject customData = new JSONObject();
customData.put("role", "admin");
sessionProperties.setCustomData(customData);

Gleap.getInstance().identifyContact("USER_ID", sessionProperties);
```

Update contact (partial updates, unsigned; fields marked "Only signed updates" stay unchanged):
```java
GleapSessionProperties props = new GleapSessionProperties();
JSONObject custom = new JSONObject();
custom.put("onboardingStep", 3);
props.setCustomData(custom);
Gleap.getInstance().updateContact(props);
```

Clear identity on logout:
```java
Gleap.getInstance().clearIdentity();
```

### Track Events

```java
Gleap.getInstance().trackEvent("User signed in");

// With data:
JSONObject data = new JSONObject();
data.put("amount", 49.99);
data.put("currency", "USD");
Gleap.getInstance().trackEvent("Purchase completed", data);
```

### Custom Data

```java
Gleap.getInstance().setCustomData("value", "key");

// Attach all at once:
JSONObject customData = new JSONObject();
customData.put("environment", "staging");
Gleap.getInstance().attachCustomData(customData);

// Remove:
Gleap.getInstance().removeCustomDataForKey("key");

// Clear all:
Gleap.getInstance().clearCustomData();
```

### Widget Control

```java
Gleap.getInstance().open();
Gleap.getInstance().close();
Gleap.getInstance().isOpened();
```
