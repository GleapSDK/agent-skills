# Cordova SDK

## Installation

```
cordova plugin add cordova-plugin-gleap
```

Works for both Android and iOS.

## Initialization

In your `deviceready` handler:

```javascript
document.addEventListener('deviceready', function () {
    cordova.plugins.GleapPlugin.initialize("API_KEY");
}, false);
```

## Common API Usage

The Cordova SDK exposes methods via `cordova.plugins.GleapPlugin`.

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start. The Cordova plugin has no `setUserJwt`; the token goes where the legacy user hash goes, as the third argument of `identify`:

```javascript
// after login and on every app start while signed in; `token` comes from your backend
cordova.plugins.GleapPlugin.identify("USER_ID", {}, token);
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout.

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```javascript
cordova.plugins.GleapPlugin.identify("USER_ID", {
  name: "Franz",
  email: "franz@example.com",
  customData: {
    role: "admin",
  },
});
```

Clear identity on logout:
```javascript
cordova.plugins.GleapPlugin.clearIdentity();
```

### Track Events

```javascript
cordova.plugins.GleapPlugin.trackEvent("User signed in");

// With data:
cordova.plugins.GleapPlugin.trackEvent("Purchase completed", {
  amount: 49.99,
  currency: "USD",
});
```

### Custom Data

```javascript
cordova.plugins.GleapPlugin.attachCustomData({ environment: "staging" });
cordova.plugins.GleapPlugin.setCustomData("key", "value");
cordova.plugins.GleapPlugin.removeCustomData("key");
cordova.plugins.GleapPlugin.clearCustomData();
```

### Widget Control

```javascript
cordova.plugins.GleapPlugin.open();
cordova.plugins.GleapPlugin.close();
```
