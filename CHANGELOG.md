# Changelog

A repository-level history of this collection: when each skill was added and when it had a notable release. Each skill also carries its own `skills/<name>/CHANGELOG.md` with full per-version detail (including `Config` notes for anything a consuming repo needs to change) — this file links out to those rather than duplicating them.

## 2026-09-12

- **loop-engineering-audit** → `v1.1.0` — new parallel-session readiness dimension, plus advisory automation-map and up-to-five loop recommendations. See [`skills/loop-engineering-audit/CHANGELOG.md`](skills/loop-engineering-audit/CHANGELOG.md).
- **repo-handoff** added (`v1.0.0`) — handoff report for a codebase inherited from another team, rolled up to Blocked/At risk/Ready with a prioritized list of questions for the previous team. See [`skills/repo-handoff/CHANGELOG.md`](skills/repo-handoff/CHANGELOG.md).
- **audit-hipaa** added (`v1.0.0`) — audits a codebase and its infrastructure config against the HIPAA Security Rule (§164.312). See [`skills/audit-hipaa/CHANGELOG.md`](skills/audit-hipaa/CHANGELOG.md).

## 2026-09-11

- **create-pr** → `v1.2.0` — splits the PR `Test plan` into `Automated` results and tickable `Manual` test scenarios. See [`skills/create-pr/CHANGELOG.md`](skills/create-pr/CHANGELOG.md).

## 2026-09-08

- **decision-record** added (`v1.0.0`) — writes or updates architecture decision records under `docs/decision-records/` in a fixed, stack-agnostic format. See [`skills/decision-record/CHANGELOG.md`](skills/decision-record/CHANGELOG.md).

## 2026-09-07

- **create-pr** added (`v1.0.0`), then bumped the same day to `v1.1.0` — opens or updates a PR with a fixed description structure, a verification checklist derived from detected tooling, and screenshots attached via `gh --attach`. See [`skills/create-pr/CHANGELOG.md`](skills/create-pr/CHANGELOG.md).
- **commit** added (`v1.0.0`) — commits everything pending as a series of grouped Conventional Commits, staged by explicit path. See [`skills/commit/CHANGELOG.md`](skills/commit/CHANGELOG.md).

## 2026-09-01

- **pr-review** → `v2.0.0` (breaking: inline GitHub review comments by default instead of a single issue comment), then `v2.1.0` (`review.language`, resolved independently of the conversation's language). See [`skills/pr-review/CHANGELOG.md`](skills/pr-review/CHANGELOG.md).

## 2026-08-29

- **pr-review** → `v1.4.1` — documents that the skill runs in any markdown-reading agent and inherits the host agent's model.

## 2026-08-28

- **mobile-store-review** added (`v1.0.0`) — audits Expo/React Native and native mobile apps against the Apple App Review Guidelines and Google Play Developer Program Policies. See [`skills/mobile-store-review/CHANGELOG.md`](skills/mobile-store-review/CHANGELOG.md).
- **loop-engineering-audit** added (`v1.0.0`) — audits a repository for readiness to be developed through autonomous agent loops. See [`skills/loop-engineering-audit/CHANGELOG.md`](skills/loop-engineering-audit/CHANGELOG.md).
- **kamal** → `v1.0.1` — shortened the frontmatter description for skills.sh, added `metadata.version` tracking. See [`skills/kamal/CHANGELOG.md`](skills/kamal/CHANGELOG.md).
- **pr-review** → `v1.3.1`, then `v1.4.0` — posts the review report as a PR comment when the scope is a GitHub PR. See [`skills/pr-review/CHANGELOG.md`](skills/pr-review/CHANGELOG.md).

## 2026-08-21

- **rest-api-design** added (`v1.0.0`) — designs and reviews REST APIs against an existing-API survey step. See [`skills/rest-api-design/CHANGELOG.md`](skills/rest-api-design/CHANGELOG.md).
- **pr-review** added (`v1.0.0`), then iterated through `v1.0.1`–`v1.3.0` the same day — evidence-based base-branch resolution, a refined severity ladder, `review.ignorePaths`, and the documentation & decision-capture lens. See [`skills/pr-review/CHANGELOG.md`](skills/pr-review/CHANGELOG.md).

## 2026-06-02

- Initial release: **kamal** skill (`v1.0.0`), MIT license, project README.
