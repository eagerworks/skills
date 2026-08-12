# Expo & React Native — Where Each Rule Lives

> Translates the checks in `permissions-and-privacy.md`, `apple-review.md`, and
> `google-play-review.md` into the actual file/config location for Expo managed, Expo with
> prebuilt native dirs, and bare React Native. Current as of Expo SDK 55 (Feb 2026, React Native
> 0.83, New Architecture default) — verify against `docs.expo.dev` if the target repo is on a much
> older or newer SDK.

## Table of Contents

1. [Workflow classification](#workflow-classification)
2. [app.config anatomy for review purposes](#appconfig-anatomy-for-review-purposes)
3. [Config plugins that inject platform config](#config-plugins-that-inject-platform-config)
4. [expo-build-properties for SDK/deployment-target floors](#expo-build-properties-for-sdkdeployment-target-floors)
5. [expo-tracking-transparency](#expo-tracking-transparency)
6. [Transitive permissions from RN libraries](#transitive-permissions-from-rn-libraries)
7. [OTA updates and the "primary purpose" rule](#ota-updates-and-the-primary-purpose-rule)
8. [Bare RN and native fallback](#bare-rn-and-native-fallback)

---

## Workflow classification

Expanded from `SKILL.md`'s Discovery step:

| Signal | Workflow | Notes |
|---|---|---|
| `app.config.ts`/`app.json`, `expo` dependency, no committed `ios/`/`android/` | **Managed (Continuous Native Generation / CNG)** | `ios/`/`android/` are generated on demand by `expo prebuild` or by EAS Build; never committed. `app.config.ts` is the only source of truth. |
| Same, **plus** committed `ios/`+`android/` | **Expo with prebuilt native dirs** | Common when a team needs custom native code beyond what config plugins support. Whether `app.config.ts` still matters depends on whether prebuild is re-run in CI on every build (`expo prebuild --clean` in a build step) or the native dirs are hand-maintained going forward. Check the CI/build script — if prebuild doesn't re-run, native dirs win and `app.config.ts` can silently drift out of sync. |
| `ios/`+`android/`, no `expo` dependency (or only `expo-modules-core` for a specific module) | **Bare React Native** | Native files are always authoritative; there's no config-plugin layer. |
| Pure Xcode/Android Studio project, no RN | **Native** | Read `Info.plist`/`AndroidManifest.xml`/`build.gradle` directly; the rest of this file doesn't apply. |

---

## app.config anatomy for review purposes

The fields that matter for a review audit, from the **resolved** output of `npx expo config --type public` (not the raw `app.config.ts` source):

```jsonc
{
  "name": "Your App",
  "version": "2.4.1",                       // marketing version — App Store Connect / Play listing
  "ios": {
    "bundleIdentifier": "com.yourcompany.yourapp",
    "buildNumber": "47",                    // must increase on every App Store Connect submission
    "infoPlist": {
      "NSCameraUsageDescription": "...",     // one entry per capability — see permissions-and-privacy.md
      "NSLocationWhenInUseUsageDescription": "...",
      "UIBackgroundModes": ["location", "audio"]
    },
    "config": {
      "usesNonExemptEncryption": false       // avoids the export-compliance prompt on every upload
    }
  },
  "android": {
    "package": "com.yourcompany.yourapp",
    "versionCode": 47,                       // must strictly increase per Play submission
    "permissions": ["CAMERA", "ACCESS_FINE_LOCATION"],
    "blockedPermissions": ["READ_EXTERNAL_STORAGE"]  // explicitly strips a permission a library would add
  },
  "runtimeVersion": { "policy": "appVersion" },       // governs OTA update compatibility — see below
  "plugins": [ /* ... */ ]
}
```

Read `ios.bundleIdentifier`/`android.package` against what's registered in App Store Connect/Play Console (not verifiable from code alone if not documented elsewhere in the repo — flag ⚪ if unconfirmable, but do flag 🔴 if it obviously contains a `.dev`/`.staging` suffix in what looks like a production profile).

## Config plugins that inject platform config

Plugins in the `plugins` array can add `Info.plist` keys, Android permissions, and native code beyond what's visible in the top-level `ios`/`android` blocks — this is the single biggest reason to always audit the *resolved* config rather than the source file.

```bash
npx expo config --type public > /tmp/resolved-config.json   # inspect the fully-merged output
npx expo-modules-autolinking search                          # which native modules actually link in
```

Common plugins that add review-relevant config: `expo-location` (location keys), `expo-camera`/`expo-image-picker` (camera/photo keys), `expo-notifications` (push entitlement, `POST_NOTIFICATIONS`), `expo-tracking-transparency` (ATT key), `expo-av`/`expo-camera` (microphone key), any custom plugin defined locally in the repo (check `plugins/` or `app.plugin.js` for hand-written `withInfoPlist`/`withAndroidManifest` mods — these are easy to miss since they're not in a published package).

## expo-build-properties for SDK/deployment-target floors

`expo-build-properties` is the managed-workflow way to set the iOS deployment target, Android `compileSdkVersion`/`targetSdkVersion`/`minSdkVersion`, and similar native build settings without ejecting:

```jsonc
✅ correct — explicit, matches current store floors
[
  "expo-build-properties",
  {
    "android": { "compileSdkVersion": 36, "targetSdkVersion": 36, "minSdkVersion": 24 },
    "ios": { "deploymentTarget": "15.1" }
  }
]
```

If this plugin is absent, the SDK/target values come from whatever the installed Expo SDK's defaults are — check those defaults against the current store floors (see `apple-review.md` / `google-play-review.md`) rather than assuming they're current; Expo usually raises its defaults to match, but not always immediately on a new floor's announcement date. In a CNG workflow, this is the **only** correct place to raise these values — hand-editing a generated `build.gradle` gets overwritten on the next `expo prebuild`.

```
❌ wrong — in a CNG workflow, edits are lost on the next prebuild
# hand-editing android/app/build.gradle directly when ios/ and android/ are NOT committed
```

## expo-tracking-transparency

If the resolved plugins list doesn't include `expo-tracking-transparency` but a tracking/ads SDK is present in the dependency tree, that's a 🔴 — see `permissions-and-privacy.md` § App Tracking Transparency.

## Transitive permissions from RN libraries

A library that links a native module can request a permission the app's own code never directly asks for. `npx expo-modules-autolinking search` lists every autolinked module; cross-reference against the permissions actually declared in the resolved config, and against each module's own native manifest snippets if it ships one (check the library's `android/src/main/AndroidManifest.xml` in `node_modules` for a definitive answer when unsure whether it declares a permission itself).

## OTA updates and the "primary purpose" rule

`expo-updates` lets JS-only changes ship without a store review. Both stores' review guidelines require that OTA-delivered updates don't change the app's primary purpose or add functionality that would itself require a fresh review (e.g. new native capabilities, a materially different core feature). Check `runtimeVersion` policy — a `runtimeVersion` mismatch between the native build and an OTA update is a technical failure, not a policy one, but is worth flagging as ⚪/note since it causes update delivery failures that look like review issues to end users.

## Bare RN and native fallback

When native files are the source of truth (bare RN, or Expo with prebuilt dirs that aren't regenerated), read directly:

```bash
# iOS
plutil -p ios/<App>/Info.plist              # macOS only; pretty-prints usage-description keys etc.
cat ios/<App>.xcodeproj/project.pbxproj | grep -E 'DEVELOPMENT_TARGET|MARKETING_VERSION|CURRENT_PROJECT_VERSION'

# Android
cat android/app/src/main/AndroidManifest.xml
cat android/app/build.gradle                # targetSdkVersion, minSdkVersion, versionCode, versionName
```

The same capability matrix in `permissions-and-privacy.md` applies — only the file you're reading from changes.
