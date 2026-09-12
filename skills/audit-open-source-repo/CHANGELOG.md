# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Audits a repository for contributor readiness across ten dimensions — discoverability & first impression, the contribution contract, community health & governance, time to first build, task surface, code orientation for feature work, guidelines as an executable spec, self-verification before the push, fork-safe CI & merge gates, and the review & release loop — graded 🔴 Barrier / 🟡 Friction / 🟢 Clear / ⚪ Unverifiable, rolled up to a mechanical verdict and an ordered, P0/P1/P2-prioritized Contributor Readiness Plan.
- A closed, exhaustive list of the conditions that may ever be graded 🔴; task surface and the review & release loop are capped dimensions that never block the verdict (`docs/decision-records/2026-09-12--task-surface-and-review-loop-are-capped-dimensions.md`).
- A first contribution dry run: an eight-stage trace derived entirely from grades already assigned, naming the first stage where a newcomer's path would break.
- Exactly one write: the report is printed in chat and saved to a dated file under `docs/open-source-audits/` in the audited repo, never overwritten (`docs/decision-records/2026-09-12--contributor-readiness-report-is-a-dated-file.md`).
- Never recommends adopting a CLA or DCO — checks consistency between what's documented and what's enforced only (`docs/decision-records/2026-09-12--never-recommend-a-contribution-agreement.md`).
- May execute the project's check-only format/lint/typecheck and test command once each to measure them, never setup/install/migrate/deploy or any fork/clone/branch/push/comment/label action.
- Copyable starters: `CONTRIBUTING.template.md`, `PULL_REQUEST_TEMPLATE.md`, two issue forms, `SECURITY.template.md`.

### Config
- New optional `.eagerworks/audit-open-source-repo.json` — `reportPath`, `language`, `project.{audience,contributionAgreement,type}`, `runCommands`, `commands.*`, `budgets.*`, `history.*`, `plan.maxItems`, `dryRun.enabled`, `dimensions.*.enabled`. See `references/config.md`.
