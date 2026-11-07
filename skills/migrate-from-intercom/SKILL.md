---
name: migrate-from-intercom
description: Migrates projects from Intercom to Gleap. Detects Intercom SDK integration (JavaScript, iOS, Android, React Native, Flutter), removes it, installs the Gleap SDK, and replaces all Intercom API calls with Gleap equivalents. Also covers server-side REST API migration. Use when switching from Intercom to Gleap, replacing Intercom, or migrating away from Intercom.
---

# Migrate from Intercom to Gleap

Guides users through replacing Intercom with Gleap in their codebase. Covers client-side SDK swap and server-side API migration.

## Detect Intercom

Search the codebase for Intercom integration. Use `Grep` to find:

### Package references
- `package.json`: `@intercom/messenger-js-sdk`, `intercom-client`, `@intercom/intercom-react-native`
- `Podfile`: `pod 'Intercom'`
- `build.gradle`: `io.intercom.android:intercom-sdk`
- `pubspec.yaml`: `intercom_flutter`
- `composer.json`: `intercom/intercom-php`
- `Gemfile`: `intercom-rails`, `intercom-ruby`

### Code patterns (client-side JavaScript — used by ALL web frameworks)
- JavaScript: `Intercom(`, `window.intercomSettings`, `widget.intercom.io`, `@intercom/messenger-js-sdk`
- iOS: `import Intercom`, `Intercom.setApiKey`, `Intercom.loginUser`
- Android: `import com.intercom`, `Intercom.initialize`, `Intercom.client()`
- React Native: `import Intercom from '@intercom/intercom-react-native'`, `IntercomModule`
- Flutter: `import 'package:intercom_flutter'`, `Intercom.instance`
- Server: `intercom-client`, `intercom-php`, `intercom-ruby`, `intercom-go`, `intercom-java`, `Intercom.Dotnet`

Report all findings to the user before proceeding.

## Project ID Resolution

The Gleap SDKs are initialized with the project's **Project ID** (formerly called "SDK key" or "API key"; same value). It is public, safe to commit and not a secret. Before asking the user for it, check these locations in order:

1. **User provided it in the conversation** — use it directly
2. **`.env` file** in the project root — look for `GLEAP_PROJECT_ID=...` (older setups: `GLEAP_API_KEY=...`, same value)
3. **Environment variable** — check `GLEAP_PROJECT_ID` (or the older `GLEAP_API_KEY`) via `echo $GLEAP_PROJECT_ID`
4. **Not found** — ask the user (Gleap dashboard at https://app.gleap.ai > **Settings → Project → General**, or **Settings → Widget → Install**)

When it is found or provided, you may offer to save it to `.env` as `GLEAP_PROJECT_ID`. Server-side code needs a different, secret credential: an API key from **Settings → Developer → API keys** (see `mapping-server-api.md`).

## Workflow

1. **Detect Intercom** using the patterns above. Report which platforms and files are affected.
2. **Confirm scope** with the user: which parts to migrate (client SDK, server API, or both).
3. **Resolve the Gleap Project ID** using the Project ID Resolution steps above.
4. **Read the mapping file** for the detected platform from this skill's directory.
5. **Remove Intercom SDK**: Uninstall packages, remove script tags, delete imports and configuration.
6. **Install Gleap SDK**: Follow the installation steps from the mapping file (or reference the `gleap-sdk-setup` skill if installed).
7. **Replace API calls**: Use the mapping tables to convert every Intercom call to its Gleap equivalent. Use the resolved Project ID in initialization code.
   - **User identity**: Intercom's `intercom_user_jwt` / `setUserJwt` becomes Gleap's identity token and `setUserJwt`; Intercom's identity verification `user_hash` becomes Gleap's legacy user hash. See "User Identity" below and re-sign on the backend with Gleap's identity key.
8. **Migrate server-side code** if applicable: Replace Intercom REST API calls with Gleap REST API calls. See `mapping-server-api.md`.
9. **Clean up**: Remove Intercom-specific config (API keys, app IDs, `intercomSettings`, push notification handlers).
10. **Verify**: Build and run the project. Search for any remaining `intercom` references.
11. **Describe the PR**: State whether identify is signed (identity token, legacy user hash or unsigned, and why), which environment variables the backend needs, and the required action in Gleap (see the end of "User Identity").

## User Identity

Intercom and Gleap prove a logged-in user the same way: the backend signs something only it can sign, the client passes it along. Map it 1:1:

| Intercom | Gleap |
|----------|-------|
| `intercom_user_jwt` (JWT signed with Intercom's identity verification secret) | Identity token: an HS256 JWT signed with one of the project's identity keys (Gleap dashboard > **Settings → Security → Customer identity**, **Create an identity key**; the secret is shown once), key id in the `kid` header, `exp` at most one hour ahead |
| `user_id` claim | `userId` claim (required) |
| `email`, `name`, `phone` claims | `email`, `name`, `phone` claims |
| Extra claims (`plan`, `company`, provider ids …) | Extra top-level claims become signed attributes on the contact; `company` is `{ "id": "...", "name": "..." }`, provider ids are `stripeCustomerId`, `chargebeeCustomerId`, `paddleCustomerId`, `lemonsqueezyCustomerId`, `revenuecatAppUserId`, `shopifyCustomerId`, `woocommerceCustomerId` |
| `Intercom.setUserJwt(jwt)` / `Intercom('boot', { intercom_user_jwt })` | `Gleap.setUserJwt(token)` after login and on app start (per-platform names in the mapping files); `clearIdentity()` on logout |
| `user_hash` (identity verification, HMAC-SHA256 of `user_id` with Intercom's secret) | Legacy user hash: hex HMAC-SHA256 of the user id with the project secret, passed where the SDK takes the user hash (third argument of `identify`). Works on existing projects while their "Accept legacy user hash" switch is on; projects created in Signed mode don't accept it, so migrate to the token |

The signing moves with the secret: replace the Intercom secret in the backend with Gleap's identity key (environment variables `GLEAP_IDENTITY_KEY_ID`, `GLEAP_IDENTITY_SECRET`), keep the claims, and the token endpoint the app already calls now returns the Gleap token. With `setUserJwt` the client passes no other contact fields; everything Gleap should know about the contact is a claim. On an installed Gleap SDK without `setUserJwt`, pass the token as the third argument of `identify(userId, data, token)`.

If the Intercom integration was unsigned (no `user_hash`, no `intercom_user_jwt`) and the app has a backend, add the token now rather than migrating the gap: Kai shows billing and account data only to signed contacts, and fields a project marks "Only signed updates" ignore unsigned writes. Plain unsigned `identify` only when there is no backend; say so in the PR. Server-side signing examples are in `mapping-server-api.md`; the `gleap-sdk-setup` skill's "User Identification" section has the full rules when it is installed.

Creating an identity key protects nothing and doesn't switch the project. New Gleap projects start in **Signed** mode and accept identity tokens only; existing (**Classic**) projects keep working as before until someone checks a token (**Check a token from your backend**) and clicks **Turn on signed sessions** on the same page. Guide: https://docs.gleap.ai/en/articles/709-customer-identity

**The PR description must state** whether identify is signed with the identity token, with the legacy user hash, or unsigned (and why), and which environment variables the backend needs. For the token, add a required action: create an identity key under Settings → Security → Customer identity, set `GLEAP_IDENTITY_KEY_ID` and `GLEAP_IDENTITY_SECRET` on the backend, check a token, and click **Turn on signed sessions** if the page still offers it.

## Mapping Files

Read the appropriate file based on detected platform:

- **JavaScript / Web**: `mapping-javascript.md` — also applies to all server-side web frameworks (Laravel, Django, Rails, ASP.NET, Spring Boot, Phoenix, etc.) since they all use the JavaScript SDK on the client side
- **iOS (Swift / Objective-C)**: `mapping-ios.md`
- **Android (Kotlin / Java)**: `mapping-android.md`
- **React Native**: `mapping-react-native.md`
- **Flutter**: `mapping-flutter.md`
- **Server-side REST API**: `mapping-server-api.md`

For projects using multiple platforms, read all relevant files. Common combinations:
- **Web framework + server SDK** (e.g., Laravel with `intercom-php`, Rails with `intercom-ruby`): Use `mapping-javascript.md` for client-side and `mapping-server-api.md` for server-side
- **React Native + server API**: Use `mapping-react-native.md` and `mapping-server-api.md`

## Important Notes

- Intercom uses `app_id` + `api_key` (separate for iOS/Android). Gleap uses a single Project ID per project (Settings → Project → General at https://app.gleap.ai) for every client SDK; server-side calls use an API key from Settings → Developer → API keys
- Intercom's `boot()` combines initialization + user identification. In Gleap, these are separate: `initialize()` then `setUserJwt()` (signed) or `identify()` (unsigned)
- Intercom's `intercom_user_jwt` maps 1:1 to Gleap's identity token (`user_id` → `userId`, extra claims → signed attributes), `Intercom.setUserJwt` → `Gleap.setUserJwt`; Intercom's identity verification `user_hash` → Gleap's legacy user hash. Re-sign on the backend with Gleap's identity key
- Intercom's `shutdown()` = Gleap's `clearIdentity()`
- Intercom's `update()` = Gleap's `updateContact()`
- Intercom custom user attributes map to Gleap's `customData` object (max 35 keys, primitives only)
- Intercom company data maps to a `company: { id, name }` claim in the identity token, or to `companyId` + `companyName` in an unsigned identify
- Features without direct Gleap equivalent: product tours (`startTour`), tickets view (`showTicket`), news items (`showNews`). Note these to the user as requiring alternative approaches
- Server-side: Intercom REST API uses `Bearer` token auth. Gleap uses `Api-Token` header (legacy) or `Bearer` + `Project` headers (v3 API)
