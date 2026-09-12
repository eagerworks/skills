# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

**Note:** version bumps were not tracked commit-by-commit before this file existed, so the entry below is a cumulative baseline covering everything shipped up to the date shown — see `git log -- skills/pr-review` for the full commit history (base-branch resolution, inline PR comments, the documentation lens, `ignorePaths`, and language configurability each landed as separate commits folded into this one entry).

## [1.0.0] - 2026-09-01

### Added
- Reviews a diff (branch, PR, staged, or working-tree changes) against a fixed five-lens rubric — correctness, security & data integrity, repo-convention conformance, test coverage, documentation & decision capture.
- Resolves the base branch from evidence rather than assuming the repo's default branch.
- Read-only by default: the standard review never edits, commits, or pushes; an optional fix loop only runs when explicitly requested.
- Posts the report to the PR under review by default when the scope is a GitHub PR — a `gh api` review with inline comments on each finding's line plus a summary body, falling back to a single `gh pr comment` when the inline post fails.
- Documentation lens (on by default) turns a diff that makes an existing doc false into a normal severity-rated finding, and an undocumented non-obvious decision into a separate, capped, non-blocking suggestion.
- Report language is deterministic and config-driven, never inferred from the language the conversation happens to be in.

### Config
- New optional `.eagerworks/pr-review.json` — all fields optional, no file required for default behavior. Schema: `baseBranch`, `review.maxRounds`, `review.extraFocus`, `review.ignorePaths` (always disclosed when it excludes a file), `review.documentation.{enabled,decisionRecordsPath,maxSuggestions}`, `review.postToPr`, `review.commentStyle` (`"inline"` default / `"summary"`), `review.language`, `localChecks`. See `references/config.md`.
