# 8. `repo-handoff` treats questions for the previous team as the primary deliverable

- **Date:** 2026-08-29

## Context

When a codebase changes hands between two development teams, the receiving
team's real problem is not what the code says — an agent can read that — but
what it *doesn't* say: who owns the Stripe account, why a migration was never
run, which cron job must not overlap, who gets paged. That knowledge lives in
the outgoing team's heads and disappears with them.

An "audit" that only lists findings pushes the work of turning gaps into
questions back onto a human, usually the night before the handoff meeting.
The result is meetings spent on things `grep` could have answered while the
important questions go unasked.

## Decision

1. The report's centrepiece is **Questions for the previous team** — numbered
   sequentially, prioritized P0/P1/P2, grouped by priority then dimension,
   each stating the evidence gap that motivates it, each with an empty
   `Answer:` line meant to be filled in during the handoff.
2. A question is emitted **only** for a check graded 🔴, 🟡, or ⚪. Asking what
   the repo already answers is a defect.
3. The report is saved to `docs/repo-handoff.md` in the analyzed repo, under
   the same rules as [ADR 7](2026-08-28--audit-report-saved-to-docs.md): one
   fixed filename, created with `docs/` if needed, overwritten on re-run,
   never committed by the skill.
4. Because the file is meant to be **filled in by humans between runs**, a
   re-run must carry over every non-empty `Answer:` line, matched by question
   text, before overwriting. Losing answers would defeat the loop.
5. Secret **values** never appear in the report or in chat — only locations
   (`file:line`, commit hash) and variable names. The skill's grep and scan
   commands are chosen to be value-blind.

## Consequences

- The verdict (Blocked / At risk / Ready) is secondary to the question list;
  a "Ready" repo with three ⚪ ownership questions still produces a report.
- The answered/open counter in the header makes the file a lightweight tracker
  for the handoff itself, and `git diff` between runs shows what got answered.
- A second write (e.g. a separate `docs/handoff-answers.md`) was rejected to
  keep the "one write" rule from ADR 7 and so answers live next to the
  evidence that prompted them.

## Related

- [`skills/repo-handoff/references/handoff-workflow.md`](../../skills/repo-handoff/references/handoff-workflow.md) — Phase 0 (recover answers) and Phase 5 (deliver).
- [`2026-08-28--audit-report-saved-to-docs.md`](2026-08-28--audit-report-saved-to-docs.md) — the one-write-to-`docs/` rule this record reuses.
