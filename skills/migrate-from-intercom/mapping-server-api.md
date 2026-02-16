# Server-Side: Intercom to Gleap API Migration

## Table of Contents
- [Remove Intercom SDKs](#remove-intercom-sdks)
- [Authentication](#authentication)
- [User/Contact Management](#usercontact-management)
- [Event Tracking](#event-tracking)
- [Conversations and Tickets](#conversations-and-tickets)
- [Node.js SDK](#nodejs-sdk)

## Remove Intercom SDKs

Remove the Intercom server SDK for your language:

| Language | Remove |
|----------|--------|
| Node.js | `npm uninstall intercom-client` |
| PHP | `composer remove intercom/intercom-php` |
| Ruby | Remove `gem "intercom-rails"` from Gemfile, run `bundle install` |
| Go | Remove `gopkg.in/intercom/intercom-go.v2` |
| Java | Remove `io.intercom:intercom-java` from pom.xml/build.gradle |
| .NET | `Uninstall-Package Intercom.Dotnet.Client` |

## Authentication

**Intercom:**
```
Authorization: Bearer <INTERCOM_TOKEN>
Intercom-Version: 2.15
```

**Gleap (legacy API):**
```
Content-Type: application/json
Api-Token: YOUR_SECRET_API_TOKEN
```

**Gleap (v3 API):**
```
Authorization: Bearer YOUR_API_KEY
Project: YOUR_PROJECT_ID
Content-Type: application/json
```

Get your Gleap API key and project ID from **Project Settings > Security > API Key** in the Gleap dashboard.

## User/Contact Management

### Identify / Create Contacts

**Before (Intercom REST):**
```bash
curl -X POST https://api.intercom.io/contacts \
  -H "Authorization: Bearer INTERCOM_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "role": "user",
    "external_id": "123",
    "email": "user@example.com",
    "name": "John Doe",
    "custom_attributes": { "plan": "Pro" }
  }'
```

**After (Gleap legacy API):**
```bash
curl -X POST https://api.gleap.io/admin/identify \
  -H "Api-Token: YOUR_SECRET_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "123",
    "email": "user@example.com",
    "name": "John Doe",
    "plan": "Pro"
  }'
```

**After (Gleap v3 API):**
```bash
curl -X POST https://api.gleap.io/v3/sessions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Project: YOUR_PROJECT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "123",
    "email": "user@example.com",
    "name": "John Doe",
    "customData": { "plan": "Pro" }
  }'
```

### Attribute mapping

| Intercom Contact | Gleap Session |
|-----------------|---------------|
| `external_id` | `userId` |
| `email` | `email` |
| `name` | `name` |
| `phone` | `phone` |
| `signed_up_at` | `createdAt` (ISO 8601) |
| `custom_attributes` | `customData` |
| `company.company_id` | `customData.companyId` |

## Event Tracking

**Before (Intercom REST):**
```bash
curl -X POST https://api.intercom.io/events \
  -H "Authorization: Bearer INTERCOM_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "event_name": "purchased-item",
    "user_id": "123",
    "created_at": 1609459200,
    "metadata": { "item": "Blue Shoes", "price": 49.99 }
  }'
```

**After (Gleap legacy API):**
```bash
curl -X POST https://api.gleap.io/admin/track \
  -H "Api-Token: YOUR_SECRET_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "events": [{
      "name": "purchased-item",
      "userId": "123",
      "date": "2021-01-01T00:00:00.000Z",
      "data": { "item": "Blue Shoes", "price": 49.99 }
    }]
  }'
```

Note: Gleap's track API accepts an array of events (batch), while Intercom sends one event per request.

## Conversations and Tickets

### Create a ticket

**Before (Intercom):**
```bash
curl -X POST https://api.intercom.io/conversations \
  -H "Authorization: Bearer INTERCOM_TOKEN" \
  -d '{ "from": { "type": "user", "id": "USER_ID" }, "body": "Help me!" }'
```

**After (Gleap v3 API):**
```bash
# Step 1: Ensure session exists
curl -X POST https://api.gleap.io/v3/sessions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Project: YOUR_PROJECT_ID" \
  -d '{ "userId": "123", "email": "user@example.com" }'

# Step 2: Create ticket (use session _id from step 1)
curl -X POST https://api.gleap.io/v3/tickets \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Project: YOUR_PROJECT_ID" \
  -d '{
    "title": "Help me!",
    "type": "BUG",
    "session": "SESSION_ID",
    "formData": { "description": "Help me!" }
  }'
```

### Add a message/comment

**Before (Intercom):**
```bash
curl -X POST https://api.intercom.io/conversations/CONV_ID/reply \
  -H "Authorization: Bearer INTERCOM_TOKEN" \
  -d '{ "message_type": "comment", "type": "admin", "admin_id": "ADMIN_ID", "body": "We are on it!" }'
```

**After (Gleap v3 API):**
```bash
curl -X POST https://api.gleap.io/v3/messages \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Project: YOUR_PROJECT_ID" \
  -d '{ "ticket": "TICKET_ID", "comment": "We are on it!" }'
```

## Node.js SDK

If using Node.js, replace the Intercom client with Gleap's admin SDK:

**Before (Intercom):**
```javascript
const { IntercomClient } = require('intercom-client');
const client = new IntercomClient({ tokenAuth: { token: 'TOKEN' } });

await client.contacts.create({ externalId: '123', email: 'user@example.com' });
await client.events.create({ eventName: 'signup', userId: '123' });
```

**After (Gleap):**
```javascript
import GleapAdmin from "gleap-admin";

GleapAdmin.initialize("SECRET_API_TOKEN");

GleapAdmin.identify("123", {
  email: "user@example.com",
  name: "John Doe",
});

GleapAdmin.trackEvent("123", "signup", {
  someData: "value",
});
```

Install: `npm install gleap-admin --save`

## Rate Limits

Both APIs enforce rate limits:
- **Intercom**: Varies by plan (most: 1000 req/min)
- **Gleap**: 1000 requests / 60 seconds per project
