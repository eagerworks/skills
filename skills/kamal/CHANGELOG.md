# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

**Note:** version bumps were not tracked commit-by-commit before this file existed, so the entry below is a cumulative baseline — see `git log -- skills/kamal` for the full commit history.

## [1.0.0] - 2026-06-30

### Added
- Deploys and troubleshoots Kamal apps, version-aware: defaults to Kamal 2.x, with all 1.9.x content isolated to `references/kamal-v1.md`.
- Opens with a version-detection step (`kamal version`, or infer from `traefik:`/`.env` → v1 vs `proxy:`/`.kamal/secrets` → v2) so it never mixes v1 syntax into v2 guidance.
- Covers first-time setup, deploys, rollbacks, `kamal-proxy` + Let's Encrypt SSL, secrets/vault adapters, accessories, builders/multiarch, and the v1→v2 upgrade path.

### Config
- No config file. Behavior adapts based on what it detects in the target repo (`config/deploy.yml`, `.kamal/secrets`, installed `kamal` version) rather than a skill-specific settings file.
