# JavaScript / Web SDK

## Table of Contents
- [Installation](#installation)
- [Initialization](#initialization)
- [Framework-Specific Notes](#framework-specific-notes)
- [Common API Usage](#common-api-usage)

## Installation

### Option A: npm / yarn (Angular, React, Vue, Next.js, Nuxt)

```
npm install gleap --save
```

### Option B: CDN Script Tag (plain HTML or any web page)

Add to `<head>`:

```html
<script>
!function(){if(!(window.Gleap=window.Gleap||[]).invoked){window.GleapActions=[];const e=new Proxy({invoked:!0},{get:function(e,n){return"invoked"===n?e.invoked:function(){const e=Array.prototype.slice.call(arguments);window.GleapActions.push({e:n,a:e})}},set:function(e,n,t){return e[n]=t,!0}});window.Gleap=e;const n=document.getElementsByTagName("head")[0],t=document.createElement("script");t.type="text/javascript",t.async=!0,t.src="https://sdk.gleap.io/latest/index.js",n.appendChild(t),
    window.Gleap.initialize("API_KEY")
}}();
</script>
```

Replace `API_KEY` with the actual key. This loads Gleap asynchronously and initializes it. No further init call needed.

## Initialization

For npm installs, import and initialize once:

```javascript
import Gleap from "gleap";

Gleap.initialize("API_KEY");
```

Call `initialize` only once. For CDN installs, initialization is included in the snippet above.

## Framework-Specific Notes

### Angular
Add initialization to `app.component.ts` or `main.ts`.

### React
Add initialization to `index.js` or `App.js` before rendering.

### Vue.js
Add initialization to `main.js` or `main.ts`.

### Next.js
Add initialization inside a `useEffect` in `_app.js` or `_app.tsx`, or in a client component.

### Nuxt
Add initialization in a client-side plugin.

### Ruby on Rails / Turbo (soft-reload)
Some stacks clear HTML on soft reload, removing the widget. Re-initialize after:

```javascript
Gleap.getInstance().softReInitialize();

// With Turbo:
document.addEventListener("turbo:load", function () {
  Gleap.getInstance().softReInitialize();
});
```

## Common API Usage

### Identify Users

Decide signed vs. unsigned with the "User Identification" section of `SKILL.md` and say in the PR which one the app uses.

**Signed (the app has a logged-in user and a backend):** the backend signs the identity token (HS256 with the project's identity key, `kid` header, claims `userId` + the contact's data, `exp` at most one hour ahead) and the app passes it after login and on app start:

```javascript
// after login and on every app start while signed in
const { token } = await fetch("/api/gleap-token", { credentials: "include" }).then((r) => r.json());
Gleap.setUserJwt(token);
```

Everything Gleap should know about the contact (email, name, company, plan, `stripeCustomerId`, …) is a claim in the token; the client passes no other contact fields. Call `clearIdentity()` on logout. On an installed SDK without `setUserJwt`, pass the token where the legacy user hash goes:

```javascript
Gleap.identify("USER_ID", {}, token);
```

**Unsigned (no backend to change):** plain identify. Say so in the PR. Never put plan, billing, company or provider ids here; the project may mark such fields "Only signed updates", and then these writes are ignored.

```javascript
Gleap.identify("USER_ID", {
  name: "Franz",
  email: "franz@example.com",
  customData: {
    role: "admin",
    logins: 42,
  },
});
```

Update contact data (partial updates, unsigned; fields marked "Only signed updates" stay unchanged):
```javascript
Gleap.updateContact({ customData: { onboardingStep: 3 } });
```

Clear identity on logout:
```javascript
Gleap.clearIdentity();
```

### Track Events

```javascript
Gleap.trackEvent("User signed in");

// With data:
Gleap.trackEvent("Purchase completed", {
  amount: 49.99,
  currency: "USD",
});
```

### Custom Data

```javascript
Gleap.attachCustomData({ environment: "staging", buildId: "abc123" });

// Incremental:
Gleap.setCustomData("key", "value");

// Remove:
Gleap.removeCustomData("key");

// Clear all:
Gleap.clearCustomData();
```

### Widget Control

```javascript
Gleap.open();
Gleap.close();
Gleap.isOpened(); // returns boolean
```
