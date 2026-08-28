# Google Play Review — Policy Catalog & Console Gates

> Covers the Play Developer Program Policies most often responsible for rejections and removals,
> plus the Play Console gates enforced before/at submission. Not a replacement for the
> [official policy center](https://play.google.com/about/developer-content-policy/) — cite the
> policy name, but verify current wording there if precision matters. Dated facts below were
> current as of August 2026; re-verify before citing a specific date, since Google revises target
> API level deadlines annually.

## Table of Contents

1. [Policy quick index](#policy-quick-index)
2. [Data safety](#data-safety)
3. [Permissions that access sensitive information](#permissions-that-access-sensitive-information)
4. [Foreground service types](#foreground-service-types)
5. [Exact alarms](#exact-alarms)
6. [Payments](#payments)
7. [Families / Designed for Families](#families--designed-for-families)
8. [User-generated content](#user-generated-content)
9. [Deceptive behavior & metadata](#deceptive-behavior--metadata)
10. [Account deletion](#account-deletion)
11. [Play Console gates](#play-console-gates)

---

## Policy quick index

| Policy | What triggers it | Where the evidence lives |
|---|---|---|
| Data safety | Declaration doesn't match what the binary actually collects/shares | SDK dependency list vs. any checked-in Data safety notes |
| Permissions & APIs (sensitive info) | Broad permission for a narrow use case | Resolved `AndroidManifest.xml` |
| Foreground service types | Foreground service without a declared, justified type | Service declarations in the manifest, API 34+ |
| Exact alarms | `SCHEDULE_EXACT_ALARM` without justification | Manifest + Play Console declaration |
| Payments | Digital goods sold outside Play Billing without meeting current external-payment terms | Payment integration code |
| Families | App in/eligible for Families program without meeting ad/data restrictions | Ad SDK config, target audience |
| UGC | No moderation tooling on a UGC surface | Feed/comment/chat screens |
| Deceptive behavior | Misleading functionality, hidden data collection | App behavior vs. declared purpose |
| Account deletion | No in-app **and** web deletion path | Settings screens + any deletion web route |

---

## Data safety

The Data safety section, filled out in Play Console, must match what the app's binary actually does. Google **actively cross-references the declaration against the installed APK/AAB** — an SDK that's linked but undeclared can result in the app being **removed**, not just a rejected update, especially for location, advertising ID, and contacts data.

Audit steps: enumerate data-collecting SDKs from the resolved dependency tree (analytics, crash reporting, ads, attribution — same list as Apple's App Privacy check, see `permissions-and-privacy.md`), and flag each as needing a corresponding Data safety declaration. If a declaration is checked into the repo, diff directly; otherwise 🟡 per undeclared-but-likely-collecting SDK.

## Permissions that access sensitive information

Google restricts several permission categories to apps whose **core function** requires them:

- **Photo & Video Permissions policy** — `READ_MEDIA_IMAGES`/`READ_MEDIA_VIDEO` (or legacy `READ_EXTERNAL_STORAGE`) should only be requested by apps whose core functionality is media-centric (galleries, backup, file managers). An app that only needs occasional picks (profile photo, attachment) should use the system photo picker intent, which needs no permission at all. See `permissions-and-privacy.md` § Photo/media permission scope.
- **`QUERY_ALL_PACKAGES`** — broad app-visibility queries need a declared, narrow use case; most apps should use package-specific queries instead.
- **`MANAGE_EXTERNAL_STORAGE`** — reserved for file-manager-class apps; requesting it for anything narrower is a near-automatic rejection.
- **Background location** (`ACCESS_BACKGROUND_LOCATION`) — requires a Play Console declaration explaining the feature, a demo video, and justification that the use is core to the app (not incidental). Google's precise-location guidance also recommends the in-app "use my location" button pattern as the minimum acceptable scope rather than requesting always-on precise location by default.
- **Contacts.** Google's Contacts Permissions policy pushes apps toward the system Contact Picker for one-off contact selection instead of requesting broad `READ_CONTACTS`; broad access needs a declared, justified core use case (e.g. a dialer or messaging app), the same pattern as Photo & Video Permissions above.

*(These bullets summarize policy directions Google has been tightening through 2026 — verify current wording and enforcement dates at [Play Console Help](https://support.google.com/googleplay/android-developer) before citing a specific compliance deadline, since Google has been rolling out several sensitive-permission and account-management policy updates on a rolling basis this year.)*

## Foreground service types

Since **Android 14 (API 34)**, every foreground service must declare a specific `android:foregroundServiceType` in the manifest and hold the matching typed permission (e.g. `FOREGROUND_SERVICE_LOCATION`, `FOREGROUND_SERVICE_CAMERA`, `FOREGROUND_SERVICE_DATA_SYNC`). Apps targeting API 34+ must also **declare the foreground service type in Play Console** (Policy → App content), with a description, user-impact statement, and — for some types — a demo video, justified by user-initiated, perceptible use.

```
✅ correct
<service android:name=".LocationTrackingService"
         android:foregroundServiceType="location" />
<!-- + FOREGROUND_SERVICE_LOCATION permission declared -->
<!-- + Play Console declaration filled out for this type -->

❌ wrong — will fail at runtime on API 34+ and blocks Play submission
<service android:name=".LocationTrackingService" />
<!-- no type declared -->
```

Flag any foreground service without a declared type as 🔴 for apps targeting API 34+.

## Exact alarms

`SCHEDULE_EXACT_ALARM` (API 31+) requires a Play Console declaration if used, justified by a use case exact alarms are actually needed for (most reminder/scheduling features can use inexact alarms or WorkManager instead). Flag if present without an obvious matching justification.

## Payments

Historically, digital goods/services sold in-app had to use Google Play Billing. This changed substantially starting **June 30, 2026**: in the **US, UK, and EEA**, developers may route payments through external payment systems (their own or third-party), and Google replaced the flat 30% commission with a decoupled fee structure — a base service fee (as low as 10% for a developer's first $1M in annual revenue) that applies regardless of billing method, plus an additional ~5% billing fee only when using Google Play's own billing system. Google will not require Play Billing exclusivity, nor prohibit communicating about alternative payment methods, in those regions. Outside those regions, check current requirements — this rollout has been region-by-region.

*(Re-verify this section before citing it — external-payment rules have changed multiple times through 2025–2026 and vary by region; don't assume a rule holds in a region not explicitly confirmed.)*

## Families / Designed for Families

Apps that target or primarily appeal to children have additional restrictions: no behavioral ads, restricted SDK list, no unencrypted transmission of certain data. If the resolved config or store category suggests a kids' audience, check ad SDK configuration for whether it's running in a "child-directed" / limited-ads mode.

## User-generated content

Same practical check as Apple's Guideline 1.2 — a report/block mechanism and content standards for any UGC surface. See `apple-review.md` § 1.2.

## Deceptive behavior & metadata

Functionality must match what's declared — a permission or background behavior not disclosed anywhere in the listing or in-app is a policy violation. Mostly overlaps with the Data safety check above; flag separately only if the app does something actively undisclosed (e.g. background data collection with no user-facing feature tied to it).

## Account deletion

Google Play's account-deletion policy is **stricter than Apple's**: if the app supports creating an account, it must offer:
1. An **in-app** path to request deletion of the account and its data.
2. A **public web URL** (reachable outside the app, e.g. linked from the privacy policy) where the same request can be made.

Deactivating, disabling, or "freezing" an account does **not** satisfy this — actual data deletion is required. Both paths must actually delete the associated user data, not just the login credential.

```
✅ correct — satisfies Play
In-app: Settings → Delete account → confirms → deletes account + data
Web: https://yourapp.example.com/delete-account (public, no login required to view the request form)

❌ wrong — satisfies neither store
Only an in-app "deactivate" toggle; no web URL
```

If sign-up exists and either the in-app path or the public web URL is missing, grade 🔴 for Play even if Apple's requirement (in-app only) is met — don't treat "account deletion: done" as one checkbox across both stores.

---

## Play Console gates

- **Target API level.** New apps and updates must target **Android 16 (API level 36)** starting **August 31, 2026** (extensions available to November 1, 2026 on request), except Wear OS and Android Automotive OS. Existing apps must target **API 35+** to remain available to new users on devices running a newer Android version than the app targets; apps targeting API 34 or lower become restricted to devices at or below that level. *(Re-verify at [Play Console Help — Target API level requirements](https://support.google.com/googleplay/android-developer/answer/11926878) — this floor moves forward roughly yearly.)*
- **Play Billing Library version.** Any app using Google Play Billing must be on **Billing Library v8.0.0 or later** for new apps and updates submitted from **August 31, 2026** (same extension window, to November 1, 2026). This is a publishing gate, not a runtime kill switch — already-published apps keep transacting, but the next update is blocked below v8. Check the version pulled in by `com.android.billingclient:billing` in `build.gradle`, or by whatever purchase library wraps it (e.g. `react-native-iap`, RevenueCat's `react-native-purchases`) — an outdated wrapper library can pin an old Billing Library version transitively. *(Re-verify at [Play Billing deprecation FAQ](https://developer.android.com/google/play/billing/deprecation-faq).)*
- **App signing.** Play App Signing should be enabled; check for a valid, non-expired signing configuration rather than a manually-managed keystore prone to loss.
- **AAB, not APK.** New apps must publish an Android App Bundle, not a raw APK.
- **`versionCode` monotonicity.** Each new release's `versionCode` must be strictly greater than the last — check `eas.json`'s `android.autoIncrement`/remote version source, or the resolved config directly.
- **Declaration forms.** Data safety, foreground service types, background location, exact alarms — see above; all live in Play Console → Policy → App content, not in code, but each has a code-visible trigger this file maps out.
