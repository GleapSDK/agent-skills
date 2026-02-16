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
| `Intercom('boot', { user_id, email, name, ... })` | `Gleap.identify(userId, { email, name, ... })` |
| `Intercom('update', { name: 'New' })` | `Gleap.updateContact({ name: 'New' })` |
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
| `user_hash` | Third argument to `identify()` |
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

**After (Gleap):**
```javascript
Gleap.identify("123", {
  email: "user@example.com",
  companyId: "c1",
  companyName: "ACME",
  plan: "Pro",
  value: 199,
  customData: { subscription_tier: "gold" },
});
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
