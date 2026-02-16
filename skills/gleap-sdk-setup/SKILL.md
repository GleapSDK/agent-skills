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

6. **Nothing detected** -> Ask the user which platform they are targeting.

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

If `package.json` is found, use `Read` to inspect its `dependencies` and `devDependencies` keys.

## Workflow

Follow these steps in order:

1. **Fetch latest SDK versions**: Run `scripts/get-latest-versions.sh` from this skill's directory. Use the returned versions in all install commands instead of hardcoded version numbers.
2. **Detect platform** using the priority rules above.
3. **Confirm with user**: State the detected platform and ask for confirmation. If the user already specified a platform, skip this step.
4. **Read platform guide**: Read the matching `platform-{name}.md` file from this skill's directory.
5. **Install SDK**: Walk the user through installing the dependency using the latest version from step 1. Run install commands when the user approves. Verify installation succeeded.
6. **Initialize SDK**: Add initialization code to the correct file. The user must provide their API key or use a placeholder `YOUR_API_KEY`. Remind them to get one at https://app.gleap.io if needed.
7. **Configure platform**: Apply required permissions, manifest entries, or additional config from the platform guide.
8. **Verify**: Suggest building/running the project to confirm integration works.

## Post-Setup API Guidance

After setup, if the user asks about using the Gleap API, refer to the "Common API Usage" section in the relevant platform file. The most common tasks are:

- **Identify users**: Associate sessions with user data (name, email, plan, custom data)
- **Track events**: Log custom events at key points in the app
- **Custom data**: Attach contextual data to feedback tickets
- **Widget control**: Programmatically open/close the Gleap widget

Re-read the platform file's API section for the correct method signatures, as they differ between platforms.

## Important Notes

- API keys are obtained free at https://app.gleap.io
- `initialize()` must be called exactly once in the application lifecycle
- For cross-platform frameworks (React Native, Flutter, Ionic/Capacitor), both iOS and Android platform-specific configuration (permissions) is needed
- The JavaScript CDN approach works for any web context and does not require npm
- For identity verification, the user hash must be generated server-side using the project's secret key
- Custom data supports only primitive values (strings, numbers, booleans) with a max of 35 keys

## Common Troubleshooting

- **`pod install` fails**: Run `pod repo update` first, then retry
- **Gradle sync fails**: Ensure `minSdkVersion` is at least 21
- **Flutter version conflict**: Add `tools:overrideLibrary="io.gleap.gleap_sdk"` to AndroidManifest.xml
- **Widget not showing on web**: Check Content Security Policy headers allow `sdk.gleap.io`
- **Soft-reload clears widget** (Rails/Turbo): Call `Gleap.getInstance().softReInitialize()` after reload
- **Android hardware acceleration**: Do not set `android:hardwareAccelerated="false"` at application level
