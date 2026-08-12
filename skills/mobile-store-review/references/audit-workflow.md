# Mobile Store Review — Audit Workflow

> The step-by-step procedure for running a full audit. For what each check actually looks for,
> see `permissions-and-privacy.md`, `apple-review.md`, `google-play-review.md`, `expo-and-rn.md`,
> and `monorepo-and-eas.md`.

## Table of Contents

1. [Phase 1 — Discover and classify](#phase-1--discover-and-classify)
2. [Phase 2 — Permissions and privacy](#phase-2--permissions-and-privacy)
3. [Phase 3 — Accounts, payments, and content](#phase-3--accounts-payments-and-content)
4. [Phase 4 — Submission gates](#phase-4--submission-gates)
5. [Phase 5 — Metadata and assets](#phase-5--metadata-and-assets)
6. [Phase 6 — Write the report](#phase-6--write-the-report)
7. [Read-only command reference](#read-only-command-reference)

---

## Phase 1 — Discover and classify

Covered in full in `SKILL.md` → Discovery. Summary:

1. Locate every mobile app in the repo (a monorepo can have more than one — e.g. a consumer app and a driver/partner app).
2. Classify each as Expo managed (CNG), Expo with committed native dirs, bare RN, or native — see `expo-and-rn.md` for the decision criteria in detail.
3. Resolve the config (`npx expo config --type public`, `npx eas config --platform <ios|android>`) rather than reading source.
4. Identify the production build profile in `eas.json` — later phases audit *that* profile's env and settings, not `development`/`preview`.

If the repo has multiple apps, produce one report per app (or one report with a clearly separated section per app) — don't merge findings across apps.

---

## Phase 2 — Permissions and privacy

Read `permissions-and-privacy.md` for the full capability matrix and store declaration rules. In this phase:

1. Enumerate every permission-requiring capability actually linked into the binary — from the resolved Expo config's `plugins`, from `android.permissions`/`ios.infoPlist` in the resolved manifest, and from the merged Android manifest / `Info.plist` when native dirs are the source of truth.
2. For each one, confirm:
   - An iOS usage-description string exists, is non-empty, and describes a specific purpose (not a placeholder like "We need this permission").
   - An Android permission is declared and — for background location, `SCHEDULE_EXACT_ALARM`, foreground service types, etc. — the corresponding Play Console declaration would be required.
   - It appears (or its category appears) in a `PrivacyInfo.xcprivacy` if it's a required-reason API.
   - It's reflected in the App Privacy questionnaire answers and the Play Data safety form, if those are present in the repo (e.g. as a checked-in JSON/notes file) — otherwise flag as ⚪ Unverifiable and note it must be checked in-console.
3. Flag anything linked but **not** declared as 🔴/🟡 depending on how likely automated detection is (see `permissions-and-privacy.md` → "declared vs. binary" mismatches).
4. Flag anything declared but not actually used — this isn't a rejection risk but is worth a 🟢/note since it's an easy cleanup.

---

## Phase 3 — Accounts, payments, and content

1. **Account deletion.** If the app has a sign-up/account-creation flow (search for register/signup screens, `POST /users`, `/signup`, `/register`, Firebase Auth `createUserWithEmailAndPassword`, etc.), confirm an in-app deletion path exists and reaches a real delete — not just a deactivate/freeze. For Play specifically, also look for a public web URL for account deletion (often referenced from a privacy policy page or a dedicated route). See `apple-review.md` § 5.1.1(v) and `google-play-review.md` § Account deletion — the two stores' requirements differ.
2. **Third-party login.** If the app offers Google/Facebook/etc. sign-in, check whether Sign in with Apple is also offered (required by Apple's Guideline 4.8 unless an exemption applies — see `apple-review.md`).
3. **Payments.** If the app sells digital goods/services or subscriptions, confirm purchases route through the platform's in-app purchase system where required, or through the mechanisms allowed under each store's current external-payment rules — see `apple-review.md` § 3.1.1 and `google-play-review.md` § Payments, both of which are storefront/region-dependent and have changed recently.
4. **UGC.** If users can post content visible to others, confirm there's a reporting/blocking mechanism and a moderation policy — Apple Guideline 1.2.
5. **Age rating / kids category.** If the app targets or could appeal to children, check for the additional data-collection and ad-network restrictions both stores apply.

---

## Phase 4 — Submission gates

These block the upload or the build, independent of review content:

- **SDK/Xcode floor (iOS)** and **target API level floor (Android)** — see the current values in `apple-review.md` and `google-play-review.md`; check the resolved `expo-build-properties` config or native `build.gradle`/`project.pbxproj` settings against them.
- **Version and build-number monotonicity** — `version`, `ios.buildNumber`, `android.versionCode` in the resolved config, or `autoIncrement`/remote version source in `eas.json`. A non-incrementing build number is a hard submit-time rejection.
- **Bundle ID / applicationId consistency** — must match what's registered in App Store Connect / Play Console; check for accidental `.dev`/`.staging` suffixes surviving into the production profile.
- **Signing & credentials** — read-only inspection only (`eas credentials` in interactive/list mode, or reading `ios/*.xcodeproj` signing settings); flag missing or expired-looking provisioning profiles, entitlements that don't match capabilities used (e.g. push notification entitlement without APNs code, or vice versa).
- **Encryption export compliance** — `ios.config.usesNonExemptEncryption` / `ITSAppUsesNonExemptEncryption` should be set explicitly rather than left for App Store Connect to ask about on every submission.

See `monorepo-and-eas.md` for how these interact with `eas.json` profiles and a Turborepo build.

---

## Phase 5 — Metadata and assets

- **Icons** — iOS icons must not have an alpha channel; check for a fully-opaque icon at the required sizes.
- **Splash/launch screen** — present and not a placeholder.
- **Support URL, marketing URL, privacy policy URL** — if these are stored in repo (e.g. in `app.config.ts` `extra` fields, or a docs/marketing site checked into the monorepo), confirm they aren't `localhost`/`example.com`/`TODO` placeholders. Reachability itself may need a live check — mark unreachable-from-code URLs ⚪.
- **Placeholder/lorem content** — grep for `TODO`, `lorem ipsum`, `test@test.com`, `Coming soon` in user-facing strings.
- **Staging config in production profile** — check the production `eas.json` profile's env vars and any `.env.production` for API URLs pointing at `staging.`/`dev.`/`localhost`, and for debug flags (`__DEV__`-gated code that should be stripped, verbose logging left on, an in-app debug menu without a guard).

---

## Phase 6 — Write the report

Fill `assets/audit-report.md`:

1. Header: app name, audit date, commit SHA, app path, resolved stack (framework + version, min/target OS versions), and an overall verdict.
2. Summary table — counts by severity, split by store.
3. 🔴 Blockers first, then 🟡 Risks, then ⚪ Unverifiable, then a 🟢 Pass summary.
4. Every finding: which store(s) it applies to, the guideline number or policy name (from `apple-review.md` / `google-play-review.md`), `file:line` evidence, a one-sentence finding, and a concrete fix.
5. Don't invent a guideline number or policy name — if unsure which rule applies, describe the issue and say so rather than guessing a citation.

---

## Read-only command reference

Safe to run — inspect state, never mutate the repo or touch store APIs:

```bash
npx expo config --type public          # resolved Expo manifest (includes plugin effects)
npx expo config --type prebuild        # resolved manifest as prebuild would apply it
npx expo-doctor                        # flags known Expo/RN misconfigurations
npx expo-modules-autolinking search    # which native modules actually link in
npx eas config --platform ios          # resolved EAS build profile (no build triggered)
npx eas config --platform android
npx eas credentials                    # inspect credentials interactively — read/list only, don't generate or revoke
npx eas build:version:get              # read current version/build numbers without changing them
```

Grep/read commands for native files when they're the source of truth:

```bash
plutil -p ios/<App>/Info.plist                       # macOS: pretty-print Info.plist
cat android/app/src/main/AndroidManifest.xml
cat android/app/build.gradle                          # targetSdkVersion / minSdkVersion / versionCode
```

**Never run:**

```bash
# ❌ mutates the repo — regenerates ios/ and android/ from app.config.ts, discarding hand edits
npx expo prebuild

# ❌ costs money, queues a real build on EAS's infrastructure
eas build

# ❌ submits to App Store Connect / Play Console — a real, possibly irreversible action
eas submit

# ❌ mutates credentials
eas credentials --clear-all
```

If a check genuinely requires one of these (e.g. confirming a build actually compiles), say so explicitly in the report as an ⚪ item and let the user decide whether to run it themselves.
