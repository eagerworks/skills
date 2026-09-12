# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Audits a codebase and its infrastructure config against the HIPAA Security Rule (45 CFR §164.312): locates PHI in data models, logs, error trackers, analytics, and outbound LLM/API calls, then reports findings graded 🔴 Blocker / 🟡 Risk / ⚪ Needs a human / 🟢 Pass with `file:line` evidence.
- Scopes first — classifies covered entity / business associate / subcontractor / neither before producing findings — and routes BAA, training, risk analysis, and breach notification to a human instead of guessing.
- Writes the graded report to `docs/hipaa-audits/YYYY-MM-DD-<slug>.md` in the audited repo.

### Config
- No config file.
