# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.1] - 2026-08-28

### Changed
- Shortened the frontmatter `description` to ~60 words (one sentence on what the skill does, one "Use when..." sentence with the highest-signal triggers) since skills.sh renders it verbatim as the page summary, `og:description`, and JSON-LD; switched `>` to `>-` to drop a trailing newline that leaked into that metadata. Verified against all 27 eval prompts across the collection before landing (24/27 matched identically; the 3 misses were mid-workflow prompts that never matched by description either way).
- Added `metadata: {author, version}` to the frontmatter, establishing this changelog's version tracking.

## [1.0.0] - 2026-06-02

Initial release.

### Added
- Deploys and troubleshoots Kamal apps, version-aware: defaults to Kamal 2.x, with all 1.9.x content isolated to `references/kamal-v1.md`.
- Opens with a version-detection step (`kamal version`, or infer from `traefik:`/`.env` → v1 vs `proxy:`/`.kamal/secrets` → v2) so it never mixes v1 syntax into v2 guidance.
- Covers first-time setup, deploys, rollbacks, `kamal-proxy` + Let's Encrypt SSL, secrets/vault adapters, accessories, builders/multiarch, and the v1→v2 upgrade path.

### Config
- No config file. Behavior adapts based on what it detects in the target repo (`config/deploy.yml`, `.kamal/secrets`, installed `kamal` version) rather than a skill-specific settings file.
