# iOS: Intercom to Gleap Migration

## Table of Contents
- [Remove Intercom](#remove-intercom)
- [Install Gleap](#install-gleap)
- [Initialization](#initialization)
- [User Identity](#user-identity)
- [Event Tracking](#event-tracking)
- [Custom Data](#custom-data)
- [Widget Control](#widget-control)

## Remove Intercom

### CocoaPods
Remove from `Podfile`:
```ruby
# DELETE: pod 'Intercom'
```
Run `pod install`.

### Swift Package Manager
In Xcode: select the Intercom package and remove it.

### Clean up
- Remove `import Intercom` from all files
- Remove `Intercom.setApiKey(...)` from AppDelegate/SceneDelegate
- Remove Intercom push notification handlers if present

## Install Gleap

### Swift Package Manager (recommended)
In Xcode: **File > Add Packages...** → `https://github.com/GleapSDK/Gleap-iOS-SDK`

### CocoaPods
Add to `Podfile`:
```ruby
pod 'Gleap', '>= LATEST_VERSION'
```
Run `pod install`.

## Initialization

Intercom uses `setApiKey` + `loginUser` as separate steps. Gleap uses `initialize` + `setUserJwt` (signed), or `identifyContact` only without a backend.

**Before (Intercom):**
```swift
import Intercom

// AppDelegate
Intercom.setApiKey("IOS_API_KEY", forAppId: "APP_ID")

// After login
let attrs = ICMUserAttributes()
attrs.userId = "123"
attrs.email = "user@example.com"
Intercom.loginUser(with: attrs) { }
```

**After (Gleap):**
```swift
import Gleap

// AppDelegate
Gleap.initialize(withProjectId: "GLEAP_PROJECT_ID")

// After login and on app start, signed (the app has a backend): the token carries userId, email, …
Gleap.setUserJwt(token)

// Only without a backend: unsigned identify
let props = GleapUserProperty()
props.email = "user@example.com"
Gleap.identifyContact("123", andData: props)
```

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom.setApiKey(key, forAppId: id)` | `Gleap.initialize(withProjectId: projectId)` (SDK 19.5.0+; older versions: `withToken:`, same value) |
| `Intercom.setUserJwt(jwt)` | `Gleap.setUserJwt(token)` (the token re-signed by your backend with Gleap's identity key; after login and on app start, no other contact fields from the client) |
| `Intercom.setUserHash(hash)` + `loginUser` | `Gleap.identifyContact(userId, andData: props, andUserHash: hash)` (legacy user hash), or better `Gleap.setUserJwt(token)` |
| `Intercom.loginUser(with: attrs)` (unsigned) | `Gleap.identifyContact(userId, andData: props)` (unsigned; only without a backend) |
| `Intercom.loginUnidentifiedUser()` | Not needed (Gleap uses guest sessions by default) |
| `Intercom.logout()` | `Gleap.clearIdentity()` |
| `Intercom.updateUser(with: attrs)` | `Gleap.updateContact(props)` |
| `Intercom.isUserLoggedIn()` | `Gleap.isUserIdentified()` |

### Attribute mapping

| Intercom (`ICMUserAttributes`) | Gleap (`GleapUserProperty`) |
|-------------------------------|----------------------------|
| `.userId` | First arg to `identifyContact` |
| `.email` | `.email` |
| `.name` | `.name` |
| `.phone` | `.phone` |
| `.companies` | `.companyId`, `.companyName` |
| `.customAttributes` | `.customData` |

**Before (Intercom):**
```swift
let attrs = ICMUserAttributes()
attrs.userId = "123"
attrs.email = "user@example.com"
attrs.name = "Franz"
attrs.customAttributes = ["plan": "Pro", "role": "admin"]
Intercom.loginUser(with: attrs)
```

**After (Gleap, signed, the app has a backend):** the backend puts these facts into the identity token (`userId`, `email`, `name`, `plan`, `role`), the app passes only the token.
```swift
Gleap.setUserJwt(token) // after login and on app start
```

**After (Gleap, unsigned, no backend):** only harmless data; the plan stays out of an unsigned identify.
```swift
let props = GleapUserProperty()
props.email = "user@example.com"
props.name = "Franz"
props.customData = ["role": "admin"]
Gleap.identifyContact("123", andData: props)
```

## Event Tracking

| Intercom | Gleap |
|----------|-------|
| `Intercom.logEvent(withName: "name")` | `Gleap.trackEvent("name")` |
| `Intercom.logEvent(withName: "name", metaData: [...])` | `Gleap.trackEvent("name", withData: [...])` |

## Custom Data

| Intercom | Gleap |
|----------|-------|
| `attrs.customAttributes = [...]` (in identify) | `props.customData = [...]` (in identify) |
| No separate custom data API | `Gleap.attachCustomData([...])` |
| No separate custom data API | `Gleap.setCustomData("val", forKey: "key")` |

## Widget Control

| Intercom | Gleap |
|----------|-------|
| `Intercom.presentMessenger()` | `Gleap.open()` |
| `Intercom.presentHelpCenter()` | `Gleap.openHelpCenter()` |
| `Intercom.presentArticle(id)` | `Gleap.openHelpCenterArticle(id)` |
| `Intercom.presentContent(.helpCenterCollections(ids))` | `Gleap.openHelpCenterCollection(id)` |
| `Intercom.setLauncherVisible(false)` | `Gleap.showFeedbackButton(false)` |

## Info.plist

Both SDKs need the same permissions. No changes needed if Intercom permissions were already added:
```xml
<key>NSPhotoLibraryUsageDescription</key>
<key>NSCameraUsageDescription</key>
<key>NSMicrophoneUsageDescription</key>
```
