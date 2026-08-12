# Apple App Review — Guideline Catalog & Submission Gates

> Covers the App Review Guidelines most often responsible for rejections, plus the submission
> gates enforced before a human reviewer ever looks at the app. Not a replacement for the
> [official guidelines](https://developer.apple.com/app-store/review/guidelines/) — cite the
> section number, but verify current wording there if precision matters. Dated facts below were
> current as of August 2026; re-verify before citing a specific date.

## Table of Contents

1. [Guideline quick index](#guideline-quick-index)
2. [1.2 — User-Generated Content](#12--user-generated-content)
3. [1.5 — Developer Information](#15--developer-information)
4. [2.1 — App Completeness](#21--app-completeness)
5. [2.3 — Accurate Metadata](#23--accurate-metadata)
6. [3.1.1 / 3.1.3 — In-App Purchase & external links](#311--313--in-app-purchase--external-links)
7. [3.1.2 — Subscriptions](#312--subscriptions)
8. [4.2 — Minimum Functionality](#42--minimum-functionality)
9. [4.8 — Login Services](#48--login-services)
10. [5.1.1 — Data Collection and Storage](#511--data-collection-and-storage)
11. [5.1.2 — Data Use and Sharing](#512--data-use-and-sharing)
12. [App Store Connect submission gates](#app-store-connect-submission-gates)
13. [Reading a rejection message](#reading-a-rejection-message)

---

## Guideline quick index

| Guideline | Name | What triggers it | Where the evidence lives |
|---|---|---|---|
| 1.2 | User-Generated Content | UGC feature with no report/block/moderation | Feed/comment/chat screens, absence of a report action |
| 1.5 | Developer Information | Broken or missing support URL | `app.config.ts` metadata, App Store Connect (not code) |
| 2.1 | App Completeness | Crashes, placeholder content, dev/staging build submitted | Build config, error boundaries, `.env.production` |
| 2.3 | Accurate Metadata | Screenshots/description don't match the app | App Store Connect listing (not code) |
| 3.1.1 / 3.1.3 | In-App Purchase / External Links | Digital goods sold outside IAP without meeting an exception | Payment integration code |
| 3.1.2 | Subscriptions | Missing subscription terms/cancellation info | Paywall screen copy |
| 4.2 | Minimum Functionality | App is a thin wrapper / too little native functionality | Whole-app judgment call — ⚪ |
| 4.8 | Login Services | Third-party login without Sign in with Apple (unless exempt) | Auth screen |
| 5.1.1 | Data Collection and Storage | Missing privacy policy; **5.1.1(v) Account Sign-In**: no in-app account deletion | Settings/account screens |
| 5.1.2 | Data Use and Sharing | Tracking without ATT consent flow | Analytics/ads SDK init code |

---

## 1.2 — User-Generated Content

Apps with UGC (posts, comments, chat, profiles other users see) need: a mechanism to report objectionable content, a way to block abusive users, published content standards, and a way to contact the developer. Search for feed/comment/chat/DM screens and check for a report/block affordance nearby. No affordance found → 🔴 for a UGC-centric app, 🟡 for a minor/incidental UGC surface (e.g. a single "leave a review" field).

## 1.5 — Developer Information

Requires a working support URL. Not verifiable from code unless the URL is defined in the repo (e.g. `app.config.ts`) — check it isn't a placeholder; actual reachability is ⚪.

## 2.1 — App Completeness

The largest rejection category by volume. Checks:
- **Placeholder content** — grep for `Lorem ipsum`, `TODO`, `test@test.com`, `Coming soon` in user-facing strings.
- **Demo account** — if login is required, App Store Connect review notes need working demo credentials, or Sign in with Apple with a note that a review account isn't needed. Not directly checkable from code; flag ⚪ with a reminder to confirm the review-notes credentials still work.
- **Build pointed at dev/staging** — see `references/audit-workflow.md` Phase 5; the production `eas.json` profile's API base URL must not point at a staging/dev host.
- **Crashes on launch** — outside static analysis; note as ⚪ unless an obvious unguarded native-module call would crash without a required permission/config.

## 2.3 — Accurate Metadata

Store listing content (screenshots, description, keywords) lives in App Store Connect, not the repo — always ⚪ from code alone. Exception: if the resolved `app.config.ts` name/description feeds the listing and contains placeholder or mismatched content, flag that.

## 3.1.1 / 3.1.3 — In-App Purchase & external links

Digital goods and services consumed within the app must generally use Apple's In-App Purchase system (StoreKit) rather than an external payment flow — physical goods/services and most "reader" app content are exempt.

**Storefront-dependent since 2025:** following a US federal court injunction, on the **US App Store storefront**, apps may include buttons, external links, or other calls to action directing users to purchase outside the app, **without needing Apple's External Purchase Link entitlement**. This does **not** apply on other storefronts, where the entitlement (or IAP) is still required for the same flow. Audit accordingly: check whether payment code branches by storefront/region, and don't flag a US-only external-payment-link flow as a violation — but do flag the same pattern shipping unconditionally to non-US storefronts.

*(Re-verify this section against current Apple guidance before citing it in a report — this area has changed multiple times and may change again.)*

## 3.1.2 — Subscriptions

Subscription apps need clear terms (length, price, auto-renewal) near the purchase point and an easy path to manage/cancel (Apple's own subscription management, since IAP subscriptions are always cancelable there). Check the paywall screen for this copy.

## 4.2 — Minimum Functionality

Apple rejects apps that are too thin — a website wrapped in a WebView with no native functionality, or content better suited to a web page. This is a holistic, subjective judgment call. **Always ⚪ Unverifiable from code** — don't grade an app 🟢 or 🔴 here; note the risk factors (e.g. "app is primarily a WebView around `<url>` — confirm it offers native-feeling functionality beyond what the website provides") without asserting a verdict.

## 4.8 — Login Services

If the app offers any third-party or social login (Google, Facebook, etc.) to set up or authenticate the *primary* account, Apple requires an equivalent privacy-respecting option — in practice, **Sign in with Apple** — unless an exemption applies: the app exclusively uses its own account system, it's an education/enterprise/business app requiring an existing institutional account, or it uses a government/industry-backed ID system.

```
✅ correct — social login offered alongside Sign in with Apple
[Continue with Google]  [Continue with Apple]  [Continue with Email]

❌ wrong — social login only, no Apple equivalent, no exemption
[Continue with Google]  [Continue with Facebook]
```

Check the auth screen for the login provider list; flag 🔴 if a third-party/social provider exists without Sign in with Apple and no clear exemption applies.

## 5.1.1 — Data Collection and Storage

Requires a privacy policy link and, per **5.1.1(v) — Account Sign-In**, an in-app path to delete the account when the app supports account creation (the guideline's official title is "Account Sign-In"; the account-deletion requirement is a sub-clause of it — don't cite it as "Guideline 5.1.1(v) — Account Deletion" verbatim, since that's not its title). A support-email-only path does not satisfy this — the deletion must be reachable and actionable inside the app itself (Google Play's requirement is stricter still — see `google-play-review.md`).

```
✅ correct
Settings → Account → "Delete my account" → confirms → calls DELETE /account

❌ wrong — satisfies neither store
Settings → Account → "Contact support to delete your account" (mailto: link only)
```

Grade 🔴 if sign-up exists and no in-app delete flow is found.

## 5.1.2 — Data Use and Sharing

Tracking (as Apple defines it — using data to target ads or share with data brokers, across apps/sites owned by other companies) requires ATT consent first. See `permissions-and-privacy.md` § App Tracking Transparency.

---

## App Store Connect submission gates

These block the upload before any human review, independent of guideline content:

- **SDK/Xcode floor.** As of **April 28, 2026**, uploads to App Store Connect must be built with **Xcode 26** or later, using an SDK for iOS 26 / iPadOS 26 / tvOS 26 / visionOS 26 / watchOS 26. This affects new submissions and updates from that date forward — it doesn't stop an already-shipped app from running on older devices. *(Re-verify at [developer.apple.com/news](https://developer.apple.com/news/) — Apple raises this floor roughly annually.)*
- **Age rating questionnaire.** Apple introduced an updated age-rating questionnaire; developers needed to submit responses by **January 31, 2026** to avoid submission interruptions. If the app hasn't been updated recently, check whether this is still pending.
- **Export compliance.** `ios.config.usesNonExemptEncryption` should be set explicitly in the resolved config rather than left for App Store Connect to ask on every submission.
- **Privacy manifest + signed SDKs.** See `permissions-and-privacy.md` — missing manifest or an unsigned listed SDK produces an `ITMS-9xxxx` upload rejection, not a review-content rejection.

---

## Reading a rejection message

When a user pastes a Resolution Center message: extract the guideline number(s) cited, map to the matching section above, then search the repo for the concrete location that caused it (the section above tells you where to look). Don't suggest appealing a rejection that traces to a real, fixable issue in the code — fix it, or if the finding is genuinely a false positive from Apple, say so explicitly with the reasoning, rather than defaulting to "you should appeal."
