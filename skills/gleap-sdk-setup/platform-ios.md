# iOS SDK

## Table of Contents
- [Installation](#installation)
- [Import](#import)
- [Initialization](#initialization)
- [Required Permissions](#required-permissions)
- [Common API Usage](#common-api-usage)

## Installation

### Option A: Swift Package Manager (recommended)

In Xcode: **File > Add Packages...** and enter:

```
https://github.com/GleapSDK/Gleap-iOS-SDK
```

Select the Gleap package and click **Add Package**.

### Option B: CocoaPods

Add to `Podfile`:

```ruby
pod 'Gleap', '>= LATEST_IOS_VERSION'
```

Replace `LATEST_IOS_VERSION` with the version from `get-latest-versions.sh`.

Then run:

```
pod install
```

Use the generated `.xcworkspace` file for all further development.

## Import

**Swift:**
```swift
import Gleap
```

**Objective-C (SPM):**
```objectivec
@import Gleap;
```

**Objective-C (CocoaPods):**
```objectivec
#import <Gleap/Gleap.h>
```

## Initialization

### UIKit (AppDelegate)

Add to `applicationDidFinishLaunchingWithOptions`:

**Swift:**
```swift
Gleap.initialize(withProjectId: "PROJECT_ID")
```

**Objective-C:**
```objectivec
[Gleap initializeWithProjectId: @"PROJECT_ID"];
```

### SwiftUI

Add to the `App` struct's `init()`:

```swift
init() {
    Gleap.initialize(withProjectId: "PROJECT_ID")
}
```

Replace `PROJECT_ID` with the Project ID (`withProjectId:` needs SDK 19.5.0 or newer; older versions take the same value as `withToken:`).

## Required Permissions

Add to `Info.plist` for image/camera attachments:

```xml
<key>NSPhotoLibraryUsageDescription</key>
<string>Allow photo access for attachments</string>

<key>NSCameraUsageDescription</key>
<string>Allow camera access for screenshots</string>
```

For audio recording:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Allow microphone access for audio feedback</string>
```

## Common API Usage

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start:

```swift
// after login and on every app start while signed in; `token` comes from your backend
Gleap.setUserJwt(token)
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout. On an installed SDK without `setUserJwt`, pass the token where the legacy user hash goes:

```swift
Gleap.identifyContact("USER_ID", andData: nil, andUserHash: token)
```

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```swift
let userProperty = GleapUserProperty()
userProperty.name = "Franz"
userProperty.email = "franz@example.com"
userProperty.customData = ["role": "admin"]

Gleap.identifyContact("USER_ID", andData: userProperty)
```

Update contact (partial updates, unsigned; fields marked "Only signed updates" stay unchanged):
```swift
let props = GleapUserProperty()
props.customData = ["onboardingStep": 3]
Gleap.updateContact(props)
```

Clear identity on logout:
```swift
Gleap.clearIdentity()
```

### Track Events

```swift
Gleap.trackEvent("User signed in")

// With data:
Gleap.trackEvent("Purchase completed", withData: ["amount": 49.99, "currency": "USD"])
```

### Custom Data

```swift
Gleap.attachCustomData(["environment": "staging", "buildId": "abc123"])

// Incremental:
Gleap.setCustomData("value", forKey: "key")

// Remove:
Gleap.removeCustomData(forKey: "key")

// Clear all:
Gleap.clearCustomData()
```

### Widget Control

```swift
Gleap.open()
Gleap.close()
Gleap.isOpened()
```
