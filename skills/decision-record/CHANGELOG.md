# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-08

Initial release.

### Added
- Writes or updates an architecture decision record as a dated markdown file under `docs/decision-records/` — `YYYY-MM-DD--kebab-slug.md`, a one-sentence declarative title with no leading number and no `Status` field, and exactly four `##` sections (Context, Decision, Consequences, Related).
- Applies the same fixed, stack-agnostic format regardless of what convention the target repo already uses, and says so out loud when the repo's existing records differ (`references/format.md` → "When the repo already has records").
- Its one write is the record itself, committed with a plain `docs:` commit — never touches the code the record is about and never opens a PR.

### Config
- No config file. The format is fixed on purpose.
