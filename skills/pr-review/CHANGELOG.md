# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [2.1.0] - 2026-09-01

### Added
- `review.language` picks the language of the report artifact (console report, inline comments, summary body, disclosure lines), with its own resolution ladder — an explicit per-run instruction, then `review.language`, then a stated `AGENTS.md`/`CLAUDE.md` convention, then the built-in default. **The language the conversation happens to be in is never an input to this ladder** — the report is a team-facing artifact posted to GitHub, not a reply to the user. The `### FINDINGS`/`### DOCUMENTATION` machine block keeps its English keys and severity enum untranslated regardless of the setting.

### Config
- New optional `review.language` in `.eagerworks/pr-review.json` — a BCP-47 tag or plain language name, default `"en"`. See `references/config.md`.

## [2.0.0] - 2026-09-01

### Changed
- **Breaking:** the default way a review posts to a PR changed from a single `gh pr comment` issue comment to an inline GitHub review (`gh api .../pulls/<N>/reviews`) that anchors each finding to its exact line, with anything that can't be anchored (no line, out-of-diff, documentation suggestions) kept in a summary body instead of dropped. A preflight anchorability check guards the atomic-POST risk, falling back to the original comment behavior automatically if the inline post fails.

### Config
- New optional `review.commentStyle` in `.eagerworks/pr-review.json` — `"inline"` (new default) or `"summary"` to keep the prior single-comment behavior. See `references/config.md`.

## [1.4.1] - 2026-08-29

### Changed
- README documents that the skill runs in any markdown-reading agent, needs only `git` (and `gh` for GitHub PRs), and inherits the host agent's model — the only model setting is the optional Claude Code subagent's frontmatter, which is called out as editable.

## [1.4.0] - 2026-08-28

### Added
- When the review targets a GitHub PR, the same markdown report shown in the console is now posted on the PR via `gh pr comment` so the whole team can read it. If `gh` is missing or unauthenticated, the report is still printed and the skill offers to post once `gh auth login` is done.

### Config
- New optional `review.postToPr` in `.eagerworks/pr-review.json` — on by default; set `false` to keep the review console-only. See `references/config.md`.

## [1.3.1] - 2026-08-28

### Changed
- Shortened the frontmatter `description` to ~60 words (one sentence on what the skill does, one "Use when..." sentence with the highest-signal triggers) since skills.sh renders it verbatim as the page summary, `og:description`, and JSON-LD; switched `>` to `>-` to drop a trailing newline that leaked into that metadata. Verified against all 27 eval prompts across the collection before landing (24/27 matched identically; the 3 misses were mid-workflow prompts that never matched by description either way).
- Added `metadata: {author, version}` to the frontmatter, establishing this changelog's version tracking.

## [1.3.0] - 2026-08-21

### Added
- New Lens 5 — **documentation & decision capture**: a diff that makes an existing `AGENTS.md`/`CLAUDE.md`/docs page assert something false is a normal severity-rated finding; an undocumented non-obvious decision is a separate, capped, non-blocking suggestion that never counts toward the verdict. On by default. The optional fix loop can draft the proposed decision record but never invents rationale it can't source.

### Config
- New optional `review.documentation.{enabled,decisionRecordsPath,maxSuggestions}` in `.eagerworks/pr-review.json`. See `references/config.md`.

## [1.2.2] - 2026-08-21

### Changed
- Defined severity markers for posted PR comments — 🔴 `critical`, 🟠 `high`, 🟡 `minor` (only `critical` had one before) — plus a most-severe-first ordering rule and the zero-findings case.

## [1.2.1] - 2026-08-21

### Changed
- Lens 4 (test coverage) now treats a skipped or disabled test (`it.skip`/`xit`/`describe.skip`/`test.todo` in Jest/Vitest; `xit`/`skip`/`pending` in RSpec) the same as a missing test — it doesn't prove an acceptance criterion any more than no test does. Also flags `.only` on a sibling test, which silently disables the rest of the file.

## [1.2.0] - 2026-08-21

### Added
- New `review.ignorePaths`: lets a repo exclude generated/vendored paths (lockfiles, generated clients, snapshots) from the full-file read the standard review otherwise requires. Any match is always disclosed in the report — never a silent skip — and a pattern that would exclude a migration or auth/scoping file is reviewed anyway and flagged as a mismatch.

### Config
- New optional `review.ignorePaths` (array of glob patterns) in `.eagerworks/pr-review.json`. See `references/config.md`.

## [1.1.1] - 2026-08-21

### Changed
- Severity ladder: split `critical`/`high`, which previously shared one bullet with no distinguishing rule. `critical` is now reserved for exploitable-now, data-loss, or cross-tenant leaks (a missing auth/tenant scope check is always `critical`); everything else in that tier is `high`.
- Lens 4: a pre-existing test whose assertions weren't updated for a changed behavior is now treated the same as a missing test for that behavior (stale-test detection).

## [1.1.0] - 2026-08-21

### Added
- Resolves the base branch from evidence instead of silently falling back to `main`/`master`/`develop`: a five-rung ladder (named branch, open PR, config, fork point, then ask) documented in `references/base-branch.md`. An open PR's actual base now outranks `.eagerworks/pr-review.json`'s `baseBranch`, and an unresolved base always asks the user rather than assuming one.

## [1.0.1] - 2026-08-21

### Fixed
- Corrected issues found by dogfooding the skill's own introductory PR: a genuine Ruby syntax error in an eval fixture, dead/inconsistent code in `SKILL.md`'s scope-detection example, wording that contradicted `CLAUDE.md`'s own no-duplication rule for the rubric, an ambiguous fix-loop stopping condition, missing language tags on code fences, and rubric examples missing their paired correct-code counterpart. Clarified that the optional Claude Code subagent needs the skill installed alongside it for its reference paths to resolve.

## [1.0.0] - 2026-08-21

Initial release.

### Added
- Reviews a diff (branch, PR, staged, or working-tree changes) against a fixed four-lens rubric — correctness, security & data integrity, repo-convention conformance, test coverage — ported from `dizenz/agent-skills`' `code-reviewer` subagent, generalized to be plugin-free and stack-agnostic (Rails and Node/TypeScript examples).
- A conservative severity ladder and markdown or machine-parseable output.
- Read-only by default: the standard review never edits, commits, or pushes; an optional fix loop only runs when explicitly requested.

### Config
- New optional `.eagerworks/pr-review.json` — all fields optional, no file required for default behavior. Schema: `baseBranch`, `review.maxRounds`, `review.extraFocus`, `localChecks`. See `references/config.md`.
