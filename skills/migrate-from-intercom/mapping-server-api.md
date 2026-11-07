# Server-Side: Intercom to Gleap API Migration

## Table of Contents
- [Remove Intercom SDKs](#remove-intercom-sdks)
- [Authentication](#authentication)
- [User/Contact Management](#usercontact-management)
- [Identity Verification to Identity Token](#identity-verification-to-identity-token)
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

**Gleap (`/admin/*` routes):**
```
Content-Type: application/json
Api-Token: YOUR_API_KEY
```

**Gleap (v3 API):**
```
Authorization: Bearer YOUR_API_KEY
Project: YOUR_PROJECT_ID
Content-Type: application/json
```

Both use the same API key: generate one under **Settings → Developer → API keys** in the Gleap dashboard (https://app.gleap.ai). One key works on every server route: the v3 API (`Authorization: Bearer` plus the `Project` header), the `/admin/*` routes (as `Api-Token`), the Segment destination and the server-to-server Conversations API. The `Project` header takes the Project ID from **Settings → Project → General** (the same ID the SDKs are initialized with). Keep the API key in an environment variable such as `GLEAP_API_TOKEN`, never in client code or the repository. The project secret is legacy: it only works while **Accept the project secret on the server API** is on, so use an API key for new code. Docs: https://docs.gleap.ai/en/articles/716-api-overview

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

**After (Gleap admin API):**
```bash
curl -X POST https://api.gleap.io/admin/identify \
  -H "Api-Token: YOUR_API_KEY" \
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

## Identity Verification to Identity Token

Intercom's identity verification signs on the backend: an HMAC-SHA256 of `user_id` with the identity verification secret (`user_hash`), or a JWT with the same secret (`intercom_user_jwt`). Gleap does the same with an identity token: an HS256 JWT signed with one of the project's identity keys (Gleap dashboard > **Settings → Security → Customer identity**, **Create an identity key**; the secret is shown once), key id in the `kid` header, `exp` at most one hour ahead. Keep the endpoint the app already calls for the Intercom value and make it return the Gleap token.

| Intercom | Gleap |
|----------|-------|
| Identity verification secret | Identity key: id in `GLEAP_IDENTITY_KEY_ID`, secret in `GLEAP_IDENTITY_SECRET` (never in the client) |
| `intercom_user_jwt` claims `user_id`, `email`, `name`, … | `userId` (required), `email`, `name`, `phone`, `company: { id, name }`, `plan`, provider ids (`stripeCustomerId`, …); every other top-level claim becomes a signed attribute |
| `user_hash` = HMAC-SHA256(`user_id`, secret) | Legacy user hash = hex HMAC-SHA256(`userId`, the project secret); accepted on existing projects while "Accept legacy user hash" is on, never on projects created in Signed mode |

**Before (Intercom, Node):**
```javascript
import crypto from "crypto";
import jwt from "jsonwebtoken";

const userHash = crypto.createHmac("sha256", process.env.INTERCOM_IDENTITY_SECRET).update(user.id).digest("hex");
// or
const intercomUserJwt = jwt.sign({ user_id: user.id, email: user.email }, process.env.INTERCOM_IDENTITY_SECRET, { expiresIn: "1h" });
```

**After (Gleap, Node):**
```javascript
import jwt from "jsonwebtoken";

const gleapToken = jwt.sign(
  {
    userId: user.id,
    email: user.email,
    name: user.name,
    company: { id: user.companyId, name: user.companyName },
    plan: user.plan,
    stripeCustomerId: user.stripeCustomerId,
  },
  process.env.GLEAP_IDENTITY_SECRET,
  { algorithm: "HS256", keyid: process.env.GLEAP_IDENTITY_KEY_ID, expiresIn: "1h" }
);
```

Any JWT library works the same way (PyJWT, firebase/php-jwt, ruby-jwt, golang-jwt, System.IdentityModel.Tokens.Jwt): HS256, `kid` header, one-hour expiry. The client passes the token with `Gleap.setUserJwt(token)` (see the platform mapping files). Omit claims the app does not have; never invent values.

**Legacy user hash (only for an existing project that accepts it, when the app must stay on `identify(userId, data, userHash)`; signed with the project secret, not an API key):**
```javascript
const userHash = crypto.createHmac("sha256", process.env.GLEAP_PROJECT_SECRET).update(user.id).digest("hex");
```

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

**After (Gleap admin API):**
```bash
curl -X POST https://api.gleap.io/admin/track \
  -H "Api-Token: YOUR_API_KEY" \
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

// An API key from Settings → Developer → API keys, read from the environment.
GleapAdmin.initialize(process.env.GLEAP_API_TOKEN);

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
