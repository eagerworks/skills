<!--
  Store Review Audit template.
  Copy this file, fill in every section, and delete this comment block.
  Grading: 🔴 Blocker · 🟡 Risk · ⚪ Unverifiable from code · 🟢 Pass — see SKILL.md § Severity Rubric.
  Every finding cites a specific Guideline number / Play policy name and a file:line — never a
  vague "this might be an issue somewhere."
-->

# Store Review Audit — <app name>

**Audited:** <YYYY-MM-DD> · **Commit:** <sha> · **App path:** `apps/<mobile-app>`
**Stack:** <Expo SDK NN (CNG) | Expo (prebuilt native) | bare RN | native> · **Targets:** iOS <min OS> / Android API <min>–<target>
**Storefronts:** <iOS App Store | Google Play | both>
**Verdict:** <🔴 Will be rejected | 🟡 Likely to pass with risk | 🟢 Ready to submit>

## Summary

| Severity | Apple | Google Play | Total |
|---|---|---|---|
| 🔴 Blocker | 0 | 0 | 0 |
| 🟡 Risk | 0 | 0 | 0 |
| ⚪ Unverifiable from code | 0 | 0 | 0 |
| 🟢 Pass | 0 | 0 | 0 |

## 🔴 Blockers — will be rejected

### [Apple] Guideline 5.1.1(v) — Account Deletion
**Evidence:** `apps/mobile/app/(tabs)/settings.tsx:42`
**Finding:** The app offers sign-up (`apps/mobile/app/auth/register.tsx:18`) but exposes no in-app account-deletion path; Settings links to a `mailto:` support address instead.
**Fix:** Add an in-app delete flow (e.g. a confirmed `DELETE /account` call) reachable from Settings.

<!-- repeat one ### block per blocker; delete this example when real findings exist -->

## 🟡 Risks — may be rejected

### [Google Play] Data safety — declaration vs. binary
**Evidence:** `apps/mobile/package.json` (via `packages/analytics`) links `<SDK name>`
**Finding:** `<SDK name>` collects `<data type>` per its own documentation; confirm the Play Console Data safety form declares this.
**Fix:** Update the Data safety form, or remove the SDK if the collection is unintentional.

<!-- repeat one ### block per risk -->

## ⚪ Unverifiable from code

- **Guideline 2.1 — demo account.** Confirm the App Store Connect review notes' credentials still work.
- **Guideline 4.2 — Minimum Functionality.** Reviewer discretion — not gradable from code.
- **Play — external URL reachability.** Confirm the privacy policy and account-deletion web URLs are live.

## 🟢 Checked and clean

- Target API level <NN> (`android/app/build.gradle:14`)
- All linked permissions have matching usage descriptions
- `PrivacyInfo.xcprivacy` present and covers all required-reason APIs in use

## Notes for the next audit

<!-- anything time-sensitive noted during this audit — e.g. "re-verify the Aug 2026 target API
     deadline before the next release," or "confirm expo-build-properties still matches the
     current SDK floor" -->
