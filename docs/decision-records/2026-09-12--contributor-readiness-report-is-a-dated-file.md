# `audit-open-source-repo`'s report is a dated file, not the siblings' fixed overwritten name

- **Date:** 2026-09-12

## Context

`loop-engineering-audit` and `repo-handoff` both write their report to a fixed, overwritten filename (`docs/loop-engineering-audit.md`, `docs/repo-handoff.md`) — the rationale in `2026-08-28--audit-report-saved-to-docs.md` is that the report is a **work plan for the project itself**, re-run after the work is done, and a fixed name lets `git diff` show the delta between runs. `audit-hipaa` instead writes a dated file under `docs/hipaa-audits/YYYY-MM-DD-<slug>.md`, never overwritten, because the report is potentially discoverable compliance evidence under a 6-year retention rule — a reason specific to HIPAA that doesn't apply here.

`audit-open-source-repo` needed its own answer, for a reason distinct from both precedents. Contributor readiness is not monotonic the way a checklist audit is: a project can be 🟢 Contributor-ready in January, add a linter and a stricter CI job in March that a fork-CI misconfiguration then breaks, and be 🔴 Closed to contributors again by June with nobody noticing, because the regression was incidental to unrelated work. A maintainer preparing to announce "we're open for contributions" wants to see the trajectory — did readiness improve steadily, or did it regress and get missed — not just the latest snapshot. `git diff` between two runs of a fixed file shows *that* two things changed; it doesn't preserve *why* the report looked the way it did the last time someone made the announcement, a case, or a decision based on it.

There's a second reason: this audit's centrepiece, the Contributor Readiness Plan, is often handed to someone other than a maintainer — a program manager staffing an open-source push, a DevRel hire whose first task is "get this repo ready," a technical writer scoping the docs backlog. That handoff moment benefits from a durable snapshot with its own date and its own PR/issue thread reference, not a file that a later run silently rewrites out from under an open conversation about it.

Two alternatives were considered and rejected:

1. **A fixed, overwritten file, like the two sibling audits.** Consistent with the house precedent for a "work plan" report. Rejected because it discards exactly the trajectory information this domain benefits from, and because a report referenced in an open PR/issue discussion could be silently rewritten mid-conversation by an unrelated re-run.
2. **No file at all, chat-only, like `pr-review`'s default.** Rejected for the same reason `audit-hipaa` rejected it: nothing durable survives the session, and a maintainer's business case for opening the project up needs something to point at.

## Decision

`audit-open-source-repo` writes its report to `docs/open-source-audits/YYYY-MM-DD-<slug>.md` in the audited repo (default slug `full-audit`), creating the `docs/open-source-audits/` directory if it doesn't exist — following `audit-hipaa`'s dated-folder convention, not the two other audits' fixed name.

1. The filename is **never overwritten**. A same-day, same-slug collision gets a numeric suffix (`-2`, `-3`).
2. This is the skill's **one write**, matching the posture of every audit in this collection — everything else stays read-only.
3. Unlike `audit-hipaa`, a re-run **does** compare against the most recent previous dated file: the report opens with a `Progress since <date>` line naming how many checks improved and how many regressed (`references/audit-workflow.md` → "Re-auditing"). The comparison is informational, layered on top of a full re-grade — the audit never trusts the old report over freshly gathered evidence, since a fix in one dimension can regress another.
4. The report is printed in full in chat before it's saved, same as every other audit here.

## Consequences

- A repo using this skill accumulates a dated readiness history in `docs/open-source-audits/`, letting a maintainer (or `git log` on that directory) see the trajectory rather than only the latest state.
- Running the skill for the first time creates a new top-level directory as a side effect of an otherwise read-only tool — `SKILL.md` and `references/audit-workflow.md` both call this out explicitly, following `audit-hipaa`'s own precedent for the same side effect.
- A report referenced in an open discussion (a tracking issue, a PR description) stays stable until someone deliberately re-runs the audit and gets a new dated file — it is never invisibly rewritten by an unrelated run.
- This collection now has three distinct answers to "where does the audit report live": `pr-review`'s chat-only default, the fixed-overwritten-file pattern (`loop-engineering-audit`, `repo-handoff`), and the dated-folder pattern (`audit-hipaa`, `audit-open-source-repo`). Each is justified independently; a future audit skill should pick deliberately and cite which precedent it follows, not default to whichever was written most recently.

## Related

- [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md) — the fixed-overwritten-file precedent this record deliberately departs from, and why.
- [2026-08-28--hipaa-audit-report-is-a-dated-repo-file](2026-08-28--hipaa-audit-report-is-a-dated-repo-file.md) — the dated-file precedent this record follows, for a different underlying reason (compliance retention vs. trajectory tracking).
- `skills/audit-open-source-repo/references/audit-workflow.md` → "Phase 7 — Deliver" and "Re-auditing" — the procedure this ADR justifies.
