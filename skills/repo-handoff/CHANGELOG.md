# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Analyzes a codebase being inherited from another team across ten dimensions — overview & architecture, environment & setup, build/test/quality, infrastructure & deploy, data, third-party services & credentials, security & access, code health & debt, process & history, operations & support — graded 🔴 Missing / 🟡 Partial / 🟢 Documented / ⚪ Unverifiable, rolled up to Blocked / At risk / Ready.
- Turns every gap into a prioritized (P0/P1/P2) list of questions for the previous team, each citing the evidence gap and carrying an `Answer:` line.
- Writes the report to `docs/repo-handoff.md` in the analyzed repo, overwritten on re-run with previously filled answers carried over.

### Config
- No config file.
