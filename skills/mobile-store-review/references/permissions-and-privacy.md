# Permissions & Privacy — Cross-Platform Reference

> Every capability an app can request touches up to four places: an Expo module or native API, an
> iOS `Info.plist` key, an Android permission, and a store privacy declaration. Miss any one of
> the four and you get a false "clean" audit. This file is the join across all four.

## Table of Contents

1. [Capability matrix](#capability-matrix)
2. [Writing usage-description strings reviewers accept](#writing-usage-description-strings-reviewers-accept)
3. [Apple PrivacyInfo.xcprivacy](#apple-privacyinfoxcprivacy)
4. [App Tracking Transparency](#app-tracking-transparency)
5. [App Privacy questionnaire vs. what's linked](#app-privacy-questionnaire-vs-whats-linked)
6. [Google Play Data safety vs. the binary](#google-play-data-safety-vs-the-binary)
7. [Photo/media permission scope](#photomedia-permission-scope)
8. [Privacy policy URL requirements](#privacy-policy-url-requirements)
9. [Common false 🟢](#common-false-)

---

## Capability matrix

| Capability | Common Expo module | iOS `Info.plist` key(s) | Android permission(s) | Extra store step |
|---|---|---|---|---|
| Location (foreground) | `expo-location` | `NSLocationWhenInUseUsageDescription` | `ACCESS_FINE_LOCATION` / `ACCESS_COARSE_LOCATION` | — |
| Location (background) | `expo-location` (background task) | `NSLocationAlwaysAndWhenInUseUsageDescription` | `ACCESS_BACKGROUND_LOCATION` | Play: background location declaration form + demo video (see `google-play-review.md`) |
| Camera | `expo-camera`, `expo-image-picker` | `NSCameraUsageDescription` | `CAMERA` | — |
| Photo library (read) | `expo-image-picker`, `expo-media-library` | `NSPhotoLibraryUsageDescription` | `READ_MEDIA_IMAGES` / `READ_MEDIA_VIDEO` (API 33+), `READ_EXTERNAL_STORAGE` (below) | Play Photo & Video Permissions policy — see below |
| Photo library (write) | `expo-media-library` | `NSPhotoLibraryAddUsageDescription` | `WRITE_EXTERNAL_STORAGE` (legacy) | — |
| Microphone | `expo-av`, `expo-camera` (video) | `NSMicrophoneUsageDescription` | `RECORD_AUDIO` | — |
| Contacts | `expo-contacts` | `NSContactsUsageDescription` | `READ_CONTACTS` | — |
| Calendar | `expo-calendar` | `NSCalendarsUsageDescription` (+ `NSCalendarsFullAccessUsageDescription` on iOS 17+) | `READ_CALENDAR` / `WRITE_CALENDAR` | — |
| Bluetooth | `expo-bluetooth` / community modules | `NSBluetoothAlwaysUsageDescription` | `BLUETOOTH_SCAN` / `BLUETOOTH_CONNECT` (API 31+) | — |
| Push notifications | `expo-notifications` | (no usage string; entitlement instead) | `POST_NOTIFICATIONS` (API 33+) | — |
| Tracking / IDFA | `expo-tracking-transparency` | `NSUserTrackingUsageDescription` | n/a (Android has no ATT equivalent — declare in Data safety instead) | Both: App Privacy "Data Used to Track You" / Data safety |
| Face ID | (implicit via auth libs) | `NSFaceIDUsageDescription` | n/a | — |
| Motion/fitness | `expo-sensors` | `NSMotionUsageDescription` | `ACTIVITY_RECOGNITION` (API 29+) | — |
| Background audio/location/fetch (foreground service) | `expo-task-manager` + background APIs | n/a | `FOREGROUND_SERVICE` + typed permission (`FOREGROUND_SERVICE_LOCATION`, etc., API 34+) | Play: foreground service type declaration — see `google-play-review.md` |
| Exact alarms | `expo-notifications` (scheduled) | n/a | `SCHEDULE_EXACT_ALARM` (API 31+) | Play: declaration required if used |

This list is representative, not exhaustive — always cross-check the resolved config (`npx expo config --type public`) rather than assuming a package needs exactly what's in this table; Expo config plugins and transitive native modules can add entries this table doesn't cover.

---

## Writing usage-description strings reviewers accept

Both Apple and Play reviewers (and Apple's automated checks) reject generic strings. The description must name the specific feature and the user benefit.

```
✅ correct
NSCameraUsageDescription = "Used to scan receipts for expense reports"

❌ wrong — generic, gets rejected or flagged
NSCameraUsageDescription = "This app needs camera access"
```

A missing or empty usage string for a linked API is not a review-time risk — it's a **hard crash at runtime on iOS** and, separately, an App Store Connect upload rejection if Apple's static analysis detects the API without the matching key. Treat it as 🔴.

---

## Apple PrivacyInfo.xcprivacy

Required since May 1, 2024 for all new/updated App Store submissions. The manifest must declare:

- **`NSPrivacyTracking`** — whether the app engages in tracking (as Apple defines it) and, if so, **`NSPrivacyTrackingDomains`**.
- **`NSPrivacyCollectedDataTypes`** — categories of data collected and their purposes.
- **`NSPrivacyAccessedAPITypes`** — every "required-reason API" the app or any linked SDK calls (e.g. `NSPrivacyAccessedAPICategoryFileTimestamp`, `...UserDefaults`, `...SystemBootTime`, `...DiskSpace`), each with one of Apple's approved reason codes.

Certain widely-used third-party SDKs (Apple maintains a published list — includes SDKs like AFNetworking, Alamofire, several Firebase and Facebook/Meta SDK components, Flutter, Capacitor, Cordova, RealmSwift, SDWebImage, and others; check the current list rather than assuming coverage) require both a privacy manifest and a cryptographic signature from the SDK vendor. If the app links one of those SDKs without an up-to-date, signed version, the upload is rejected at App Store Connect with an `ITMS-9105x`/`ITMS-9106x`-family error (the specific code varies by which requirement is missing — manifest vs. signature vs. a declared-but-unused reason) — this is a submission gate, not a review-content issue, and it will not surface until upload time. See [Apple's third-party SDK requirements](https://developer.apple.com/support/third-party-SDK-requirements/) for the current list.

Audit steps:
1. Confirm `PrivacyInfo.xcprivacy` exists (in the app target, or per-module in dependencies) — see `assets/PrivacyInfo.xcprivacy` for a starter.
2. Check whether any dependency is on Apple's signature-required SDK list, and if so, confirm the vendored version is signed (check the SDK's own release notes/changelog for "privacy manifest" or "signed").
3. Cross-check `NSPrivacyAccessedAPITypes` entries against what the resolved native modules actually call — a required-reason API used without a declared reason is a 🔴.

---

## App Tracking Transparency

If the app links any SDK that does cross-app/cross-site tracking (most ad and some analytics SDKs), it must:
- Declare `NSUserTrackingUsageDescription`.
- Call the ATT prompt (`expo-tracking-transparency`'s `requestTrackingPermissionsAsync`, or equivalent) before accessing the IDFA or engaging in tracking, **not** just at random launch.
- Respect a "denied" response — verify the code path doesn't silently track anyway when the user declines.

Missing the ATT prompt while shipping a tracking SDK is a 🔴 — Apple's App Privacy cross-check and its static analysis both catch this pattern routinely.

---

## App Privacy questionnaire vs. what's linked

The App Store's "nutrition label" (App Privacy) is filled out in App Store Connect, not in the codebase — so the codebase can't confirm the *declared answers* are correct, only whether the *binary's behavior* would justify a given answer. Report format: "The binary links `<SDK>`, which collects `<data type>` per its own privacy manifest / documentation — confirm the App Privacy questionnaire in App Store Connect declares this," graded 🟡 unless the repo has some artifact recording the actual declared answers (then compare directly and grade 🔴/🟢).

---

## Google Play Data safety vs. the binary

Same shape as App Privacy, but Google actively cross-references the Data safety form against the installed APK/AAB's declared permissions and (increasingly) SDK signatures — an undeclared-but-linked data-collecting SDK is treated as a policy violation, not just a listing inaccuracy, and can result in app removal, not merely a rejected update.

Audit steps:
1. List every SDK in the resolved dependency tree known to collect data (analytics, crash reporting, ads, attribution).
2. For each, note what it collects per its own documentation.
3. If a Data safety declaration is present in the repo (e.g. as a checked-in reference doc), diff against the list; otherwise flag 🟡 "confirm Data safety form matches" per SDK found.

---

## Photo/media permission scope

Both stores penalize requesting broad photo-library access when the feature only needs one-off picks.

```
✅ correct — one-off image pick, no persistent library access needed
import * as ImagePicker from 'expo-image-picker'
// uses the system picker UI; no READ_MEDIA_IMAGES needed on iOS 14+/Android 13+
// via the native photo/document picker, which runs out-of-process

❌ wrong — requests full library permission for a single upload
requestMediaLibraryPermissionsAsync()   // broad grant
// ...then only ever reads one photo the user just took
```

Google's Photo & Video Permissions policy specifically restricts `READ_MEDIA_IMAGES`/`READ_MEDIA_VIDEO` to apps whose **core** functionality requires broad media access (galleries, backup apps, file managers) — an app that only needs occasional picks should use the system picker intent instead, which doesn't require the permission at all. Flag broad media permissions on apps whose primary function isn't media-centric as 🟡 (Play may require a declaration form and a justification video, or reject broad access outright depending on the declared use case).

---

## Privacy policy URL requirements

Both stores require a live, reachable privacy policy URL — not just present in App Store Connect/Play Console, but reachable and accurately describing current data practices. If the URL is defined in the codebase (e.g. `app.config.ts` `extra.privacyPolicyUrl`, or a marketing site checked into the monorepo), confirm it isn't a placeholder (`example.com`, `TODO`, `localhost`); actual reachability is outside static analysis — mark ⚪ if it can't be confirmed from the repo alone.

---

## Common false 🟢

- **Permission removed from JS, still in the native manifest.** Someone deleted `expo-camera` from `package.json` but the committed `android/app/src/main/AndroidManifest.xml` still declares `CAMERA` (from before native dirs were last regenerated). The binary still requests it; Data safety still needs to cover it.
- **SDK vendored via a workspace package, not the app's own `package.json`.** A `packages/analytics` internal package pulls in a tracking SDK; auditing only `apps/mobile/package.json` misses it entirely. Always resolve the full dependency graph, not just the app package's direct deps.
- **`app.config.ts` says one thing, `eas.json`'s production profile env overrides it.** A plugin conditionally added based on `process.env.APP_VARIANT` can silently not run in the profile that actually ships.
