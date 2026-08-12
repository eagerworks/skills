---
name: mobile-store-review
description: >
  Audits a mobile app codebase — Expo/React Native (managed or bare) or native iOS/Android,
  standalone or inside a Turborepo/monorepo — and reports whether it would pass Apple App Review
  and Google Play review. Use this skill whenever the user: asks if their app will pass, fail, or
  get rejected by Apple or Google review; is preparing an App Store or Play Store submission; got
  a rejection email or resolution center message citing a guideline or policy; is filling out the
  App Privacy questionnaire or Play Data safety form; is auditing permissions, usage descriptions,
  a privacy manifest, or account-deletion flow; mentions App Review Guidelines, Play Developer
  Program Policies, TestFlight, `eas submit`, App Store Connect, or Play Console; or is doing a
  pre-submission or release-readiness check on a mobile app. Also use when the user says things
  like "will this get rejected?", "audit my app before I ship it", or "review my app for the
  stores" — even if they don't name a specific guideline.
---

# Mobile Store Review Skill

Audits a mobile app's source and config — not the running app, not the store listing — and reports whether Apple App Review and Google Play review would pass it. Most first-submission rejections are compliance plumbing that's visible in code: a missing usage description, no in-app account-deletion path, a target SDK below the current floor, a privacy declaration that doesn't match what's actually linked. This skill finds those before a human reviewer does.

## Discovery — Do This First

Locate the app and classify its stack before checking anything else.

**1. Find the mobile app(s).** In a monorepo, check `turbo.json` for app-shaped tasks (`build`, `ios`, `android`) and the workspace globs in `pnpm-workspace.yaml` / `package.json#workspaces` / `turbo.json#packages`. Within candidate packages, look for `app.json`, `app.config.js`/`app.config.ts`, an `expo` dependency, or committed `ios/` + `android/` directories. A monorepo can contain more than one mobile app — audit each separately.

**2. Classify the workflow** — this determines which file is the *source of truth*:

| Signal | Workflow | Source of truth |
|---|---|---|
| `app.config.*`/`app.json`, no committed `ios/`/`android/` | Expo managed (CNG) | The resolved Expo config |
| `app.config.*`/`app.json` **and** committed `ios/`+`android/` | Expo + prebuilt native dirs | The native files — `app.config.ts` may be stale |
| `ios/`+`android/`, no `expo` dependency | Bare React Native | The native files |
| No RN/Expo — pure `ios/`/`android/` project | Native | The native files |

If both `app.config.ts` and committed native directories exist, **the native directories win** — a permission or usage description added only to `app.config.ts` never reaches the shipped binary unless `expo prebuild` regenerates the native dirs from it. Don't assume; check whether the native dirs are regenerated in CI or hand-maintained.

**3. Resolve the real config.** `app.config.ts` is JavaScript and can branch on environment — never audit the literal source text as if it were the final manifest. Read-only commands to resolve it:

```bash
npx expo config --type public        # fully resolved manifest, including plugin effects
npx expo-doctor                      # flags known misconfigurations
npx eas config --platform ios        # resolved build profile (which profile is production)
npx eas config --platform android
```

**4. Record the target.** Which storefronts (iOS App Store, Google Play, both)? Which `eas.json` build profile ships to production? That's the profile whose config the audit should follow — a `development` or `preview` profile's env vars are not relevant.

## What This Skill Does NOT Do

Static analysis cannot see: server-side behavior, real crash rates, the actual App Store Connect / Play Console listing (screenshots, description, category), whether a review demo account currently works, or purely subjective calls (Guideline 4.2 Minimum Functionality, 4.3 Spam, "quality" bars). Findings in those areas are graded ⚪ **Unverifiable from code**, never 🟢 Pass — a report that silently skips its blind spots is worse than one that names them.

## Severity Rubric

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | Hard, mechanical trigger — upload rejected or review rejection near-certain (missing usage description for a linked API, target SDK below the enforced floor, sign-up with no in-app deletion path) |
| 🟡 **Risk** | Reviewer-discretion or a mismatch that's often but not always caught (Data safety vs. binary, permission scope, a login wall) |
| 🟢 **Pass** | Checked against a concrete rule and clean |
| ⚪ **Unverifiable from code** | Needs a human, a store console, or a live build to confirm |

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| Running the audit end-to-end; the exact command sequence | `references/audit-workflow.md` |
| Permissions, usage descriptions, ATT, privacy manifest, App Privacy / Data safety | `references/permissions-and-privacy.md` |
| Citing an App Review Guideline; App Store Connect submission gates | `references/apple-review.md` |
| Citing a Play policy; Play Console declarations and the target API level floor | `references/google-play-review.md` |
| Where a rule lives in Expo config vs. bare RN vs. native files | `references/expo-and-rn.md` |
| Finding the app in a monorepo; EAS build/submit, versioning, credentials | `references/monorepo-and-eas.md` |

Copyable assets live in `assets/`:
- `assets/audit-report.md` — the graded report template; fill this in as the deliverable
- `assets/PrivacyInfo.xcprivacy` — starter Apple privacy manifest with required-reason API categories
- `assets/pre-submission-checklist.md` — a copyable human checklist for a release runbook, covering the ⚪ items code can't verify

## The Audit in Six Phases

See `references/audit-workflow.md` for the full procedure and exact commands.

1. **Discover & classify** — locate the app(s), determine the workflow, resolve the config (above).
2. **Permissions & privacy** — every linked capability has a usage description, a privacy manifest entry, and matches its store declaration (`references/permissions-and-privacy.md`).
3. **Accounts, payments & content** — account deletion, third-party login, IAP vs. external payment links, UGC moderation, age rating.
4. **Submission gates** — SDK/Xcode floor, target API level, version/build-number monotonicity, bundle ID / applicationId consistency, signing.
5. **Metadata & assets** — icons, splash, support/privacy-policy URL reachability, placeholder content, staging URLs baked into the production profile.
6. **Write the report** — fill `assets/audit-report.md`: Blockers, then Risks, then Unverifiable, then a Pass summary, each with `file:line` evidence and a fix.

## Critical Gotchas

1. **Read the resolved config, not the file.** `app.config.ts` can branch on `process.env`; grepping the source misses env-conditional plugins entirely.
   ```bash
   # ✅ correct
   npx expo config --type public
   # ❌ wrong — misses plugin effects and env branches
   cat app.config.ts
   ```

2. **Committed `ios/`+`android/` dirs beat `app.config.ts`.** If they exist and aren't regenerated in CI, they are what ships. A usage description or permission present only in `app.config.ts` is a false 🟢 if the native files are stale.

3. **A transitive dependency can add a permission you never asked for.** A workspace package can pull in an SDK that links an API requiring `NSPhotoLibraryUsageDescription` even though the app's own code never touches the photo library. Audit resolved native modules (`npx expo-modules-autolinking` / the merged Android manifest), not just direct `package.json` dependencies.

4. **Sign-up implies account deletion — with different requirements per store.** Apple's Guideline 5.1.1(v) ("Account Sign-In") is satisfied by an in-app deletion flow. Google Play additionally requires a **public web URL** for account deletion, declared in the Data safety form — a support email is not sufficient for either store.

5. **Never run `eas build`, `eas submit`, or `expo prebuild`.** `prebuild` mutates the repo (regenerates `ios/`/`android/`); `build`/`submit` cost money and touch store APIs. This is a read-only audit — see `references/audit-workflow.md` for the full allowed-command list.

6. **Declared privacy must match the binary — both stores now cross-reference this.** An analytics or crash-reporting SDK that's linked but undeclared in App Privacy / Data safety is a removal risk, not a minor note.

7. **Don't grade subjective guidelines 🟢.** Guideline 4.2 (Minimum Functionality), 4.3 (Spam), and similar reviewer-judgment rules go in ⚪ Unverifiable, never Pass — code can't confirm them.

8. **Facts here go stale.** SDK floors, target API levels, and billing rules change on published deadlines. Before citing a specific date or version number in a report, verify it's still current — see `references/apple-review.md` and `references/google-play-review.md` for the values known at time of writing, each flagged with when to re-check.
