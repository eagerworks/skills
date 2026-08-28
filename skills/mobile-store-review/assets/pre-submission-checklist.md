<!--
  Copyable pre-submission checklist — drop into a release runbook or PR template.
  Pairs with the audit report: items here are mostly the ⚪ Unverifiable-from-code items an
  automated audit can't confirm, plus a few code-checkable items worth a final human glance.
-->

## Pre-submission checklist

### Both stores

- [ ] Ran the `mobile-store-review` audit against the production build profile; no open 🔴 Blockers
- [ ] Privacy policy URL is live and describes current data practices
- [ ] Support URL is live
- [ ] App icon renders correctly at all required sizes, no placeholder/lorem content anywhere
- [ ] Production API base URL confirmed (not staging/dev)
- [ ] Version/build number incremented from the last submitted release

### Apple App Store

- [ ] Review demo account credentials (if login is required) tested and working right now
- [ ] `PrivacyInfo.xcprivacy` present and matches actual data collection + required-reason API usage
- [ ] App Privacy questionnaire in App Store Connect matches what the binary actually does
- [ ] Built with the current required Xcode/SDK version (check `references/apple-review.md` for the current floor)
- [ ] Age rating questionnaire answers up to date
- [ ] If sign-up exists: in-app account deletion works end-to-end
- [ ] If third-party login exists: Sign in with Apple offered alongside it (or a documented exemption applies)
- [ ] Screenshots and description in App Store Connect match current app behavior

### Google Play

- [ ] Data safety form in Play Console matches what the binary actually collects/shares
- [ ] Target API level meets the current Play floor (check `references/google-play-review.md`)
- [ ] Foreground service types (if any) declared in Play Console with description + demo video
- [ ] Background location / exact alarms (if used) declared with justification
- [ ] If sign-up exists: in-app **and** public web account-deletion paths both work
- [ ] App Bundle (AAB) built and signed via Play App Signing
- [ ] Store listing screenshots and description match current app behavior

### Final gate

- [ ] Someone other than the author has reviewed this checklist
- [ ] The `mobile-store-review` audit report is attached to the release PR/runbook entry
