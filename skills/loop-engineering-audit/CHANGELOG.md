# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.1.0] - 2026-09-12

### Added
- New **parallel-session readiness** dimension: can several agent sessions work the repo at once, in separate `git worktree`s, without colliding. Graded like the other seven dimensions, but never rolls up to a 🔴 — it caps throughput, not the verdict (`docs/decision-records/2026-09-12--parallel-session-readiness-is-a-capped-dimension.md`).
- New **advisory loop recommendations**: an automation map classifying ten delivery stages, plus up to five recommended agent loops from a ten-loop catalog (`references/loop-catalog.md`), each with a trigger, a machine-checkable done signal, prerequisites expressed as rubric check ids, a blast-radius risk level (L1/L2/L3), guardrails, and an owner/cadence/kill-switch line. Purely advisory — adds no Work Plan rows, changes no grade, never moves the verdict (`docs/decision-records/2026-09-12--loop-recommendations-are-advisory.md`).

### Config
- New `.eagerworks/loop-engineering-audit.json` keys: `dimensions.parallelSessions.enabled` (to disable the new dimension) and `loops.{enabled,maxRecommended,maxRiskLevel}` (to turn recommendations off or cap them — default `maxRiskLevel: 2`, so L3/production-reaching loops are never proposed unless raised). Both are optional; omitting them keeps prior behavior except for the two additions above. See `references/config.md`.

## [1.0.0] - 2026-08-28

Initial release.

### Added
- Audits a repository for loop-engineering readiness across seven dimensions — agent-facing context, reproducible environment, fast deterministic verification, test coverage, task definition surface, CI & merge gates, guardrails — graded 🔴 Blocker / 🟡 Gap / 🟢 Ready / ⚪ Unverifiable, rolled up to a mechanical verdict and an ordered Work Plan.
- Exactly one write: the report is printed in chat and saved to `docs/loop-engineering-audit.md` in the audited repo (fixed name, overwritten, never committed).
- May execute the project's lint/typecheck/test/build once to measure them, never setup/migrate/install/deploy commands.

### Config
- New optional `.eagerworks/loop-engineering-audit.json` — `reportPath`, `commands.*`, `budgets.*`, `runCommands`, `dimensions.*.enabled`. See `references/config.md`.
