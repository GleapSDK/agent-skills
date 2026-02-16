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

Intercom uses `setApiKey` + `loginUser` as separate steps. Gleap uses `initialize` + `identifyContact`.

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
Gleap.initialize(withToken: "GLEAP_API_KEY")

// After login
let props = GleapUserProperty()
props.email = "user@example.com"
Gleap.identifyContact("123", andData: props)
```

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom.setApiKey(key, forAppId: id)` | `Gleap.initialize(withToken: key)` |
| `Intercom.loginUser(with: attrs)` | `Gleap.identifyContact(userId, andData: props)` |
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

**After (Gleap):**
```swift
let props = GleapUserProperty()
props.email = "user@example.com"
props.name = "Franz"
props.plan = "Pro"
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
