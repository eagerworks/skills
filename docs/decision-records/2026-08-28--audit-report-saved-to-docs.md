# `loop-engineering-audit` persists its report to `docs/` in the audited repo

- **Date:** 2026-08-28

## Context

The review-style skills in this collection are read-only: `pr-review` never
creates a file in the target repo (its only mutation is a PR comment), and
`mobile-store-review` fills a report template but leaves where it goes to the
user. That's the right default for a *review* — the diff is the artifact, and
the findings belong next to it.

`loop-engineering-audit` is different in kind. Its output is a **work plan for
the project itself** — the ordered list of changes needed before agents can
develop the repo in unattended loops. That list:

- outlives the chat session it was produced in (the work takes days, not
  minutes, and is often done by someone other than whoever ran the audit);
- is re-run after the work is done, so successive reports should be
  **diffable** in git to show what improved;
- is the kind of thing the audited repo's own `docs/` is for — the same place a
  `CLAUDE.md`, an ADR, or a runbook would live.

Leaving it only in the chat transcript throws that away; asking the user to
copy-paste it every time is the same outcome with extra steps.

## Decision

1. The audit **always** writes its full report to `docs/loop-engineering-audit.md`
   at the audited repo's root, creating `docs/` if it doesn't exist. The path
   can be moved with `reportPath` in `.eagerworks/loop-engineering-audit.json`
   but not disabled.
2. The filename is **fixed, not dated** — the file is overwritten on every run
   so the project carries exactly one current audit and `git diff` shows the
   delta between runs. History lives in git, not in a pile of dated files.
3. The report is **printed in full in chat first**, then saved. The file is a
   copy of the deliverable, not a substitute for it.
4. That file is the skill's **only** write. It is left unstaged — never
   `git add`ed, committed, or pushed — and nothing else in the repo is
   created, edited, or deleted. Executing the project's lint/typecheck/test/
   build commands to measure them is allowed under the rules in
   `references/audit-workflow.md`; setup, migration, install, and deploy
   commands are never run.

## Consequences

- The skill's `SKILL.md` gotcha #1 reads "one write, nothing else" instead of
  `pr-review`'s "read-only" — reviewers of future changes should hold that
  line: no second file, no auto-commit.
- A repo that `.gitignore`s `docs/` gets told so in the report rather than
  worked around.
- This collection's own `docs/` is project documentation; running the audit on
  this repo produces a `docs/loop-engineering-audit.md` that should not be
  committed here unless we actually want to track our own readiness.

## Related

- [`skills/loop-engineering-audit/references/audit-workflow.md`](../../skills/loop-engineering-audit/references/audit-workflow.md) — Phase 5, delivery.
- [`2026-08-21--optional-fix-loop-and-round-cap.md`](2026-08-21--optional-fix-loop-and-round-cap.md) — `pr-review`'s read-only default, which this record deliberately deviates from.
