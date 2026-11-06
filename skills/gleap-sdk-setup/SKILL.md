---
name: gleap-sdk-setup
description: Integrates the Gleap customer feedback SDK into projects. Detects the platform (JavaScript, iOS, Android, React Native, Flutter, Ionic Capacitor, Cordova, FlutterFlow) and guides through installation, initialization, permissions, and common API usage like user identification and event tracking. Use when adding Gleap, setting up feedback SDK, or integrating Gleap SDK.
---

# Gleap SDK Setup

Guides users through integrating the Gleap SDK into their project. Supports JavaScript, iOS, Android, React Native, Flutter, Ionic/Capacitor, Cordova, and FlutterFlow.

## Platform Detection

Detect the user's platform before proceeding. If the user explicitly states their platform, skip detection.

### Auto-Detection Priority

Check in this order (first match wins):

1. **`pubspec.yaml` exists?**
   - Contains `flutterflow` references or user mentions FlutterFlow -> **FlutterFlow** (`platform-flutterflow.md`)
   - Otherwise -> **Flutter** (`platform-flutter.md`)

2. **`package.json` exists?** Read it and check `dependencies` + `devDependencies`:
   - `react-native` present -> **React Native** (`platform-react-native.md`)
   - `@capacitor/core` or `capacitor-core` present -> **Ionic/Capacitor** (`platform-ionic-capacitor.md`)
   - `cordova` present, or `config.xml` exists with `<widget>` -> **Cordova** (`platform-cordova.md`)
   - `@angular/core` present -> **JavaScript** (Angular)
   - `react` present (without `react-native`) -> **JavaScript** (React)
   - `vue` present -> **JavaScript** (Vue)
   - `next` present -> **JavaScript** (Next.js)
   - `nuxt` present -> **JavaScript** (Nuxt)
   - Other or no framework -> **JavaScript** (generic npm)
   - Use `platform-javascript.md` for all JavaScript variants

3. **`*.xcodeproj`, `*.xcworkspace`, or `Podfile` exists?** -> **iOS** (`platform-ios.md`)

4. **`build.gradle`, `build.gradle.kts`, or `app/build.gradle` exists?** -> **Android** (`platform-android.md`)

5. **`index.html` exists (no `package.json`)?** -> **JavaScript** CDN approach (`platform-javascript.md`)

6. **Server-side web framework detected?** These all use the **JavaScript SDK** on the client side (`platform-javascript.md`):
   - `composer.json` exists (PHP / Laravel) — use CDN or npm depending on whether a JS build pipeline exists
   - `Gemfile` with `rails` (Ruby on Rails) — use CDN in layouts, or npm if Webpacker/esbuild/importmap is present
   - `requirements.txt` / `pyproject.toml` with `django` or `flask` (Python) — use CDN in base templates
   - `*.cshtml` / `*.csproj` files (.NET / ASP.NET) — use CDN in layout views
   - `pom.xml` / `build.gradle` with Spring Boot (Java) — use CDN in Thymeleaf templates
   - Any other web framework (Go, Elixir/Phoenix, etc.) — use CDN approach
   - Note: If the project also has a `package.json` with a frontend framework, step 2 already covers this.

7. **Nothing detected** -> Ask the user which platform they are targeting.

### Detection Commands

Use `Glob` to scan for key files:
- `pubspec.yaml`
- `package.json`
- `**/*.xcodeproj`
- `**/*.xcworkspace`
- `Podfile`
- `build.gradle`
- `build.gradle.kts`
- `app/build.gradle`
- `config.xml`
- `index.html`
- `composer.json`
- `Gemfile`
- `requirements.txt`
- `pyproject.toml`

If `package.json` is found, use `Read` to inspect its `dependencies` and `devDependencies` keys.

## API Key Resolution

Before asking the user for their API key, check these locations in order:

1. **User provided it in the conversation** (e.g., "add Gleap with token abc123") — use it directly
2. **`.env` file** in the project root — look for `GLEAP_API_KEY=...`
3. **Environment variable** — check if `GLEAP_API_KEY` is set via `echo $GLEAP_API_KEY`
4. **Not found** — ask the user to provide their API key (available at https://app.gleap.io under Project Settings > Security > API Key)

When a key is found or provided, offer to save it to the project's `.env` file (creating it if needed, and adding `.env` to `.gitignore` if not already there) so it's available for future use.

## Workflow

Follow these steps in order:

1. **Fetch latest SDK versions**: Run `scripts/get-latest-versions.sh` from this skill's directory. Use the returned versions in all install commands instead of hardcoded version numbers.
2. **Detect platform** using the priority rules above.
3. **Resolve API key** using the API Key Resolution steps above.
4. **Confirm with user**: State the detected platform and ask for confirmation. If the user already specified a platform, skip this step.
5. **Read platform guide**: Read the matching `platform-{name}.md` file from this skill's directory.
6. **Install SDK**: Walk the user through installing the dependency using the latest version from step 1. Run install commands when the user approves. Verify installation succeeded.
7. **Initialize SDK**: Add initialization code using the resolved API key.
8. **Identify users**: If the app has a logged-in user, wire the signed identity token (see "User Identification" below) with the platform's `setUserJwt` call from the platform guide. Plain unsigned `identify` only when there is no backend.
9. **Configure platform**: Apply required permissions, manifest entries, or additional config from the platform guide.
10. **Verify**: Suggest building/running the project to confirm integration works.
11. **Describe the PR**: State whether identify is signed (identity token, legacy user hash or unsigned, and why) and which environment variables the backend needs.

## User Identification

Decide this once, before writing an identify call, and state the result in the PR description.

### The app has a logged-in user (the normal case): signed identity token

Gleap trusts what the backend signs and nothing the browser or app says about itself. The flow:

1. **The backend signs an identity token**: an HS256 JWT signed with one of the project's identity keys (Gleap dashboard > Settings > Security > Identity keys; the secret is shown once). Put the key id in the `kid` header and the secret in an environment variable. The secret never reaches the client.
   - Claims: `userId` (required, the same id the app uses for the user), optional `email`, `name`, `phone`, `company` (`{ "id": "...", "name": "..." }`), `plan`, provider ids such as `stripeCustomerId`, `chargebeeCustomerId`, `paddleCustomerId`, `revenuecatAppUserId`, `shopifyCustomerId`. Every further top-level claim (string, number or boolean) becomes a signed attribute on the contact.
   - `exp`: required for new integrations, at most one hour ahead. Omit claims the app does not have; never invent values.
2. **Expose it on an authenticated endpoint** of the app's backend (for example `GET /api/gleap-token`) that returns the token for the signed-in user.
3. **The app calls `setUserJwt(token)`** after login and on every app start while a user is signed in, and `clearIdentity()` on logout. Everything Gleap should know about the contact is a claim in the token; the client passes no other contact fields.
4. **Token refresh (optional)**: register `setUserJwtProvider` with a function that fetches a fresh token; the SDK calls it when a token has expired.

| Platform | Call |
|----------|------|
| JavaScript | `Gleap.setUserJwt(token)` |
| iOS | `Gleap.setUserJwt(token)` |
| Android | `Gleap.getInstance().setUserJwt(token)` |
| React Native | `Gleap.setUserJwt(token)` |
| Flutter | `Gleap.setUserJwt(token: token)` |
| Ionic/Capacitor | `Gleap.setUserJwt({ token })` |
| Cordova | No `setUserJwt`; pass the token as the third argument of `identify(userId, data, token)` |
| FlutterFlow | A Custom Action that calls the Flutter method |

**Installed SDK without `setUserJwt`** (older version, Cordova): pass the token where the legacy user hash goes, as the third argument of `identify(userId, data, token)` (per-platform shape in the platform guide). Suggest updating the SDK in the PR so `setUserJwt` can be used.

Backend example (Node, `npm install jsonwebtoken`; any JWT library in any language works the same way: HS256, `kid` header, one-hour expiry):

```javascript
import jwt from "jsonwebtoken";

export function gleapIdentityToken(user) {
  return jwt.sign(
    {
      userId: user.id,
      email: user.email,
      name: user.name,
      company: { id: user.companyId, name: user.companyName },
      plan: user.plan,
      stripeCustomerId: user.stripeCustomerId,
    },
    process.env.GLEAP_IDENTITY_KEY_SECRET,
    { algorithm: "HS256", keyid: process.env.GLEAP_IDENTITY_KEY_ID, expiresIn: "1h" }
  );
}
```

Environment variables the backend needs: `GLEAP_IDENTITY_KEY_ID` (the key id, shown in the dashboard) and `GLEAP_IDENTITY_KEY_SECRET` (the secret, shown once on creation). Add them to `.env.example` with empty values, never commit real values.

### No backend to change: plain unsigned identify

Only when the app has no backend (static site, prototype, a client-only app): call `identify(userId, data)` without a token. Say so in the PR: the identity is unverified, Kai shows no billing or account data to such contacts, and fields the project marks "Only signed updates" (Settings > Contacts > Attributes) ignore these writes. Never put plan, billing, company or provider ids in an unsigned identify; they belong in the token.

### Legacy user hash

Existing integrations may pass a hex HMAC-SHA256 of the user id, computed server-side with the project's secret API key, as the user hash (third argument of `identify`). It keeps working while the project's "Accept legacy user hash" switch is on. New integrations use the identity token; when migrating, replace the hash endpoint with the token endpoint and the `identify(..., hash)` call with `setUserJwt(token)`.

### What the PR description must say

- Whether identify is **signed with the identity token**, **signed with the legacy user hash**, or **unsigned**, and why.
- Which **environment variables** the backend needs (for the token: `GLEAP_IDENTITY_KEY_ID`, `GLEAP_IDENTITY_KEY_SECRET`).

## Post-Setup API Guidance

After setup, if the user asks about using the Gleap API, refer to the "Common API Usage" section in the relevant platform file. The most common tasks are:

- **Identify users**: Signed identity token via `setUserJwt` when the app has a logged-in user (see "User Identification"); plain `identify` only without a backend
- **Track events**: Log custom events at key points in the app
- **Custom data**: Attach contextual data to feedback tickets
- **Widget control**: Programmatically open/close the Gleap widget

Re-read the platform file's API section for the correct method signatures, as they differ between platforms.

## Important Notes

- `initialize()` must be called exactly once in the application lifecycle
- For cross-platform frameworks (React Native, Flutter, Ionic/Capacitor), both iOS and Android platform-specific configuration (permissions) is needed
- The JavaScript CDN approach works for any web context and does not require npm
- User identification: the identity token is signed on the backend with the project's identity key and passed with `setUserJwt`; the legacy user hash (HMAC of the user id with the project's secret API key) is also computed server-side and keeps working while "Accept legacy user hash" is on. Never sign or hash on the client
- Custom data supports only primitive values (strings, numbers, booleans) with a max of 35 keys; attributes that gate data or billing (plan, provider customer ids, company) go into the identity token's claims, not into unsigned custom data

## Common Troubleshooting

- **`pod install` fails**: Run `pod repo update` first, then retry
- **Gradle sync fails**: Ensure `minSdkVersion` is at least 21
- **Flutter version conflict**: Add `tools:overrideLibrary="io.gleap.gleap_sdk"` to AndroidManifest.xml
- **Widget not showing on web**: Check Content Security Policy headers allow `sdk.gleap.io`
- **Soft-reload clears widget** (Rails/Turbo): Call `Gleap.getInstance().softReInitialize()` after reload
- **Android hardware acceleration**: Do not set `android:hardwareAccelerated="false"` at application level
