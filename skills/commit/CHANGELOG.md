# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-07

Initial release.

### Added
- Commits everything modified or new in the working tree by default, split into a series of Conventional Commits grouped by the ladder in `references/grouping.md` (staged index → explicit instruction → change intent → mechanical companions → everything else).
- Infers the type/scope vocabulary from the repo's own `git log` before falling back to the full Conventional Commits set.
- Hard exclusion list applied regardless of config: `.env`, credentials, `node_modules`, build output, screenshots.

### Config
- New optional `.eagerworks/commit.json` — all fields optional, no file required for default behavior. Schema: `commit.types`, `commit.scopes`, `commit.requireScope`, `commit.subjectMaxLength`, `commit.language`, `commit.includeUntracked` (`"always"` default / `"ask"` / `"never"`), `commit.maxCommitsPerRun`, `commit.signoff`, `commit.trailers`. See `references/config.md`.
