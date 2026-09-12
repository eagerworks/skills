# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

**Note:** version bumps were not tracked commit-by-commit before this file existed, so the entry below is a cumulative baseline covering everything shipped up to the date shown — see `git log -- skills/create-pr` for the full commit history.

## [1.0.0] - 2026-09-11

### Added
- Opens or updates a PR with a fixed description structure (Summary → Problem → Solution → Screenshots/Demo → Test plan → Checklist), resolving the base branch from evidence rather than assuming the repo's default branch (`references/base-branch.md`).
- Derives the verification checklist from tooling actually detected in the repo.
- Attaches screenshots via `gh`'s `--attach` flag (≥ 2.99.0) when the change is UI-visible — never fabricates a screenshot URL or commits one into the repo.
- Splits `Test plan` into `### Automated` (real results) and tickable `### Manual` scenarios for the reviewer, on by default — never fabricates a scenario, credential, or step the diff doesn't support.
- Mutates by design: commits pending work, pushes, creates/edits the PR — the one deliberate write-posture exception in this repo (see `docs/decision-records/2026-09-07--create-pr-write-posture.md`).

### Config
- New optional `.eagerworks/create-pr.json` — all fields optional, no file required for default behavior. Schema: `baseBranch`, `pr.titleFormat`, `pr.assignSelf`, `pr.labels`, `pr.draft`, `pr.reviewers`, `pr.language`, `pr.sections`, `pr.checklist`, `pr.screenshots.{mode,uiPaths,artifactPaths,captureCommand}`, `pr.manualTestCases.{enabled,maxScenarios}` (manual test cases on by default, uncapped unless `maxScenarios` is set). See `references/config.md`.
