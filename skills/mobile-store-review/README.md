# mobile-store-review

A portable agent skill that audits a mobile app codebase — Expo/React Native (managed or bare) or native iOS/Android, standalone or inside a Turborepo/monorepo — and reports whether it would pass Apple App Review and Google Play review. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Locating a mobile app inside a Turborepo/monorepo and classifying its workflow (Expo managed, Expo with prebuilt native dirs, bare RN, native)
- Permissions, iOS usage-description strings, `PrivacyInfo.xcprivacy` and required-reason APIs, App Tracking Transparency
- App Privacy vs. Google Play Data safety — declared vs. what's actually linked
- Account deletion, third-party login (Sign in with Apple / Guideline 4.8), in-app purchases vs. external payment links
- UGC moderation, kids/families rules, age ratings, placeholder/metadata checks
- Submission blockers: SDK/Xcode floors, target API level, versioning, signing & credentials
- Expo config resolution, config plugins, `eas.json`, `.easignore`, Metro/workspace hoisting
- A severity-graded audit report (🔴 Blocker / 🟡 Risk / ⚪ Unverifiable from code / 🟢 Pass) citing the exact guideline number or Play policy name with `file:line` evidence

## Scope and limits

This is a **static, read-only audit**. It reads config and source and runs read-only commands (`npx expo config`, `npx expo-doctor`, `npx eas config`) — it never runs `eas build`, `eas submit`, or `expo prebuild`, and it never touches App Store Connect or Play Console. Some things genuinely can't be checked from code — a live demo account, the actual store listing, subjective guidelines like Minimum Functionality — and the report marks those ⚪ Unverifiable rather than guessing.

Store rules change on published deadlines. The reference files date every volatile fact (SDK floors, target API levels, policy changes) and flag it for re-verification — treat a cited date as "current when written," not permanent.

## Layout

```
SKILL.md                          # hub: discovery step, severity rubric, routing table, gotchas (agent entrypoint)
references/
  audit-workflow.md               # the 6-phase audit procedure + exact read-only commands
  permissions-and-privacy.md      # cross-platform capability matrix, privacy manifest, ATT, Data safety
  apple-review.md                 # App Review Guideline catalog + App Store Connect submission gates
  google-play-review.md           # Play Developer Program Policy catalog + Play Console gates
  expo-and-rn.md                  # where each rule lives in Expo config vs. bare RN vs. native
  monorepo-and-eas.md             # finding the app in a Turborepo, EAS build/submit, versioning, credentials
assets/
  audit-report.md                 # severity-graded report template — the audit's output shape
  PrivacyInfo.xcprivacy           # annotated starter Apple privacy manifest
  pre-submission-checklist.md     # copyable human checklist for a release runbook
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill mobile-store-review
```
