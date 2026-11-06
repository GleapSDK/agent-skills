# JavaScript / Web: Intercom to Gleap Migration

## Table of Contents
- [Remove Intercom](#remove-intercom)
- [Install Gleap](#install-gleap)
- [Initialization](#initialization)
- [User Identity](#user-identity)
- [Event Tracking](#event-tracking)
- [Custom Data](#custom-data)
- [Widget Control](#widget-control)
- [Callbacks](#callbacks)
- [Features Without Direct Equivalent](#features-without-direct-equivalent)

## Remove Intercom

### npm package
```
npm uninstall @intercom/messenger-js-sdk
```

### Script tag
Remove the Intercom loader snippet from `<head>`:
```html
<!-- DELETE: Intercom script tag containing widget.intercom.io -->
```

### Clean up
- Delete `window.intercomSettings` object
- Remove all `Intercom(...)` calls

## Install Gleap

```
npm install gleap --save
```

Or use CDN (add to `<head>`):
```html
<script>
!function(){if(!(window.Gleap=window.Gleap||[]).invoked){window.GleapActions=[];const e=new Proxy({invoked:!0},{get:function(e,n){return"invoked"===n?e.invoked:function(){const e=Array.prototype.slice.call(arguments);window.GleapActions.push({e:n,a:e})}},set:function(e,n,t){return e[n]=t,!0}});window.Gleap=e;const n=document.getElementsByTagName("head")[0],t=document.createElement("script");t.type="text/javascript",t.async=!0,t.src="https://sdk.gleap.io/latest/index.js",n.appendChild(t),
    window.Gleap.initialize("YOUR_GLEAP_API_KEY")
}}();
</script>
```

## Initialization

Intercom combines init + user login in `boot()`. Gleap separates them.

**Before (Intercom):**
```javascript
Intercom('boot', {
  app_id: 'YOUR_APP_ID',
  user_id: '12345',
  email: 'user@example.com',
  name: 'John Doe',
  created_at: 1609459200,
});
```

**After (Gleap):**
```javascript
import Gleap from "gleap";

// Step 1: Initialize (once, on app load)
Gleap.initialize("YOUR_GLEAP_API_KEY");

// Step 2: Identify user (on login/signup)
Gleap.identify("12345", {
  email: "user@example.com",
  name: "John Doe",
  createdAt: new Date(1609459200 * 1000),
});
```

## User Identity

| Intercom | Gleap |
|----------|-------|
| `Intercom('boot', { intercom_user_jwt: token })` | `Gleap.setUserJwt(token)` (the token re-signed by your backend with Gleap's identity key; no other contact fields from the client) |
| `Intercom('update', { intercom_user_jwt: newToken })` | `Gleap.setUserJwt(newToken)` |
| `Intercom('boot', { user_id, user_hash, ... })` (identity verification) | `Gleap.identify(userId, { ... }, userHash)` with the legacy user hash, or better `Gleap.setUserJwt(token)` |
| `Intercom('boot', { user_id, email, name, ... })` (unsigned) | `Gleap.identify(userId, { email, name, ... })` (unsigned; only without a backend) |
| `Intercom('update', { name: 'New' })` | `Gleap.updateContact({ name: 'New' })` (unsigned; fields marked "Only signed updates" stay unchanged) |
| `Intercom('shutdown')` | `Gleap.clearIdentity()` |
| `Intercom('getVisitorId')` | No equivalent (Gleap uses guest sessions) |

### Attribute mapping

| Intercom attribute | Gleap property |
|-------------------|----------------|
| `user_id` | First argument to `identify()` |
| `email` | `email` |
| `name` | `name` |
| `phone` | `phone` |
| `created_at` (unix timestamp) | `createdAt` (Date object) |
| `intercom_user_jwt` | `Gleap.setUserJwt(token)`; claims map `user_id` → `userId`, `email`/`name`/`phone` unchanged, `company` → `company: { id, name }`, every other claim → a signed attribute |
| `user_hash` | Third argument to `identify()` (legacy user hash, HMAC-SHA256 of the user id with Gleap's secret API key; works while "Accept legacy user hash" is on) |
| `company.company_id` | `companyId` |
| `company.name` | `companyName` |
| `company.plan` | `plan` |
| `company.monthly_spend` | `value` |
| Custom attributes (flat keys) | `customData: { key: value }` |

**Before (Intercom):**
```javascript
Intercom('boot', {
  app_id: 'APP_ID',
  user_id: '123',
  email: 'user@example.com',
  company: { company_id: 'c1', name: 'ACME', plan: 'Pro', monthly_spend: 199 },
  subscription_tier: 'gold',
});
```

**After (Gleap, signed, the app has a backend):** the backend signs the same facts into the identity token, the client passes only the token.
```javascript
// backend (Node): HS256 with the project's identity key, kid header, 1 h expiry
const token = jwt.sign(
  { userId: "123", email: "user@example.com", company: { id: "c1", name: "ACME" }, plan: "Pro", value: 199, subscription_tier: "gold" },
  process.env.GLEAP_IDENTITY_KEY_SECRET,
  { algorithm: "HS256", keyid: process.env.GLEAP_IDENTITY_KEY_ID, expiresIn: "1h" }
);

// client, after login and on app start
Gleap.setUserJwt(token);
```

**After (Gleap, unsigned, no backend):** only harmless data; plan, spend and company stay out of an unsigned identify.
```javascript
Gleap.identify("123", {
  email: "user@example.com",
  customData: { subscription_tier: "gold" },
});
```

**Identity verification (`intercom_user_jwt` / `user_hash`):**

**Before (Intercom):**
```javascript
Intercom('boot', { app_id: 'APP_ID', intercom_user_jwt: jwtFromYourBackend });
// or, older: Intercom('boot', { app_id: 'APP_ID', user_id: '123', user_hash: hmacFromYourBackend });
```

**After (Gleap):**
```javascript
Gleap.setUserJwt(tokenFromYourBackend); // same claims, re-signed with Gleap's identity key
// legacy hash, only while "Accept legacy user hash" is on: Gleap.identify("123", {}, hmacFromYourBackend);
```

## Event Tracking

| Intercom | Gleap |
|----------|-------|
| `Intercom('trackEvent', 'name')` | `Gleap.trackEvent('name')` |
| `Intercom('trackEvent', 'name', { key: 'val' })` | `Gleap.trackEvent('name', { key: 'val' })` |

Direct 1:1 mapping. No changes needed beyond the method name.

## Custom Data

Intercom passes custom data as flat keys in `boot`/`update`. Gleap uses separate methods:

| Intercom | Gleap |
|----------|-------|
| Flat keys in `intercomSettings` | `Gleap.attachCustomData({ key: value })` |
| `Intercom('update', { key: value })` | `Gleap.setCustomData('key', 'value')` |

## Widget Control

| Intercom | Gleap |
|----------|-------|
| `Intercom('show')` | `Gleap.open()` |
| `Intercom('hide')` | `Gleap.close()` |
| `Intercom('showMessages')` | `Gleap.openConversations()` |
| `Intercom('showNewMessage', 'text')` | `Gleap.startConversation()` |
| `Intercom('showArticle', id)` | `Gleap.openHelpCenterArticle(id)` |
| `Intercom('showSpace', 'help')` | `Gleap.openHelpCenter()` |
| `Intercom('showSpace', 'news')` | `Gleap.openNews()` |
| `Intercom('showSpace', 'messages')` | `Gleap.openConversations()` |

## Callbacks

| Intercom | Gleap |
|----------|-------|
| `Intercom('onShow', fn)` | `Gleap.on('open', fn)` |
| `Intercom('onHide', fn)` | `Gleap.on('close', fn)` |
| `Intercom('onUnreadCountChange', fn)` | `Gleap.on('unread-count-changed', fn)` |

## Features Without Direct Equivalent

| Intercom feature | Gleap alternative |
|-----------------|-------------------|
| `startTour(tourId)` | Use Gleap product tours (configured in dashboard) |
| `startSurvey(surveyId)` | `Gleap.showSurvey(surveyId)` |
| `startChecklist(checklistId)` | Gleap checklists (configured in dashboard) |
| `showTicket(ticketId)` | No direct equivalent |
| `showConversation(convId)` | `Gleap.openConversation(convId)` |
| `showNews(newsItemId)` | `Gleap.openNewsArticle(newsItemId)` |
| `custom_launcher_selector` | `Gleap.open()` on click of your custom element |
| `alignment` / `horizontal_padding` / `vertical_padding` | Configure in Gleap dashboard widget settings |
