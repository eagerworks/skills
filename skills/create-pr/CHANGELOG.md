# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.2.0] - 2026-09-11

### Added
- Splits the `Test plan` section into `### Automated` (unchanged) and `### Manual` subsections, so whoever reviews the PR gets tickable, self-contained scenarios (setup, steps, expected result) instead of only author-side command output. Applies to any observable behavior, not just UI, and never fabricates a step, credential, or scenario the diff doesn't support — an unknown becomes `TODO(author):`, and a change with no observable surface gets a one-line reason instead of the subsection.

### Fixed
- `pr.manualTestCases.maxScenarios` defaults to no cap instead of an arbitrary 5, which was dropping real manual test cases the diff warranted.
- A step naming the feature/route being unlocked is kept even when the account detail needed to reach it is `TODO(author)`'d, instead of silently collapsing to a generic "sign in" step.

### Config
- New optional `pr.manualTestCases.{enabled,maxScenarios}` in `.eagerworks/create-pr.json` — on by default, uncapped unless `maxScenarios` is set. See `references/config.md`.

## [1.1.0] - 2026-09-07

### Added
- `references/screenshots.md` and `SKILL.md` gotcha 6 now explicitly forbid committing a screenshot into the repo's git history as a workaround for not having `gh --attach` (e.g. adding it to `screenshots/` and linking via a `raw.githubusercontent.com` URL) — that permanently adds a review-only binary to the project's history for a problem `--attach` already solves.
- `pr.language` gained a per-run override rung ahead of the config file, matching `pr-review`'s `review.language` ladder: an explicit instruction in the user's request for this run wins once, without touching `.eagerworks/create-pr.json`.

## [1.0.0] - 2026-09-07

Initial release.

### Added
- Opens or updates a PR with a fixed description structure (Summary → Problem → Solution → Screenshots/Demo → Test plan → Checklist), resolving the base branch from evidence rather than assuming the repo's default branch (`references/base-branch.md`).
- Derives the verification checklist from tooling actually detected in the repo.
- Attaches screenshots via `gh`'s `--attach` flag (≥ 2.99.0) — the first officially supported way to get an image into a PR body via automation — when the change is UI-visible; never fabricates a screenshot URL or commits one into the repo.
- Asks before overriding a repo's own PR template rather than silently picking either side.
- Mutates by design: commits pending work, pushes, creates/edits the PR — the one deliberate write-posture exception in this collection (see `docs/decision-records/2026-09-07--create-pr-write-posture.md`).

### Config
- New optional `.eagerworks/create-pr.json` — all fields optional, no file required for default behavior. Schema: `baseBranch`, `pr.titleFormat`, `pr.assignSelf`, `pr.labels`, `pr.draft`, `pr.reviewers`, `pr.language`, `pr.sections`, `pr.checklist`, `pr.screenshots.{mode,uiPaths,artifactPaths,captureCommand}`. See `references/config.md`.
