# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-08-28

Initial release.

### Added
- Audits an Expo/React Native (managed or bare) or native iOS/Android codebase, standalone or inside a Turborepo/monorepo, against the Apple App Review Guidelines and Google Play Developer Program Policies.
- Covers permissions & usage descriptions, privacy manifests & App Tracking Transparency, App Privacy vs. Data safety, account deletion, IAP & external payments, SDK/target-API floors, versioning & credentials, and EAS/monorepo build config.
- Outputs a severity-graded audit report.

### Config
- No config file.
