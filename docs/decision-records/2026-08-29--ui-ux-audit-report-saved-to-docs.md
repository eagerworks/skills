# `ui-ux-audit` persists its report to `docs/` and recommends on every item

- **Date:** 2026-08-29

## Context

`ui-ux-audit` is the second audit-style skill in the collection, after
`loop-engineering-audit`. It grades a project's interface across eight
dimensions and produces a work plan for designers and developers. Two
questions came up when shaping it:

1. **Where does the report go?** [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md) decided that an *audit* (as opposed
   to a *review*) is a work plan for the project itself — it outlives the chat,
   is re-run after the work, and should be diffable in git. A UI/UX audit has
   the same shape: the fixes take days, are often done by someone other than
   whoever ran it, and a re-audit should show what improved.
2. **Does every item need a recommendation, or only the failing ones?**
   `loop-engineering-audit` writes a fix only for 🔴/🟡 checks and lists 🟢
   checks as bare ids. That is fine for an engineering audience that reads a
   grade as an instruction. A UI/UX audit is read by designers, PMs, and
   developers who did not run it and who need to know *what to do* about each
   line — including what to keep doing, and how to verify what the audit could
   not see (contrast, focus rings, reflow) without a running app.

The user's request for this skill was explicit: the output "must have clear
recommendations of what to do in every item."

## Decision

1. The audit **always** writes its full report to `docs/ui-ux-audit.md` at the
   audited repo's root, creating `docs/` if needed. `reportPath` in
   `.eagerworks/ui-ux-audit.json` can move it but not disable it. Fixed
   filename, overwritten on every run, printed in full in chat first, left
   unstaged — the same four rules as [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md).
2. **Every check gets a `→ Recommendation:` line, regardless of grade.**
   🔴/🟡 name the file or component and the pattern to apply; 🟢 start with
   **Keep:** and name the practice that earned the grade; ⚪ start with
   **Verify:** and give the exact URL/viewport/element or question, and what
   result would make it 🟢 vs 🟡/🔴. A check with a grade but no
   recommendation is an incomplete finding and the report is not delivered
   until it has one (`references/rubric.md` → Recommendation rule).
3. The Work Plan still maps 1:1 to 🔴/🟡 checks. The recommendation rule adds
   text to the findings, not rows to the plan — a Solid verdict still has an
   empty Work Plan.
4. The report is the skill's **only** write. It never edits a view, component,
   stylesheet, or locale file. The optional browser pass is bounded: it may
   navigate, resize, tab, screenshot, and read computed styles on an app that
   is **already running**; it never starts a server, installs dependencies,
   submits forms that create real data, or clicks destructive actions
   (`references/audit-workflow.md` → Phase 2).

## Consequences

- `ui-ux-audit` findings are longer than `loop-engineering-audit` findings by
  one line per 🟢 check; consecutive 🟢 checks that share a practice may share
  one **Keep:** line to keep it readable.
- Re-audits can see regressions in what was previously 🟢, because the
  **Keep:** lines record what earned the grade.
- A future audit-style skill should adopt both rules unless it has a reason
  not to; a review-style skill (`pr-review`, `mobile-store-review`) stays
  read-only per [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md).

## Related

- [`2026-08-28--audit-report-saved-to-docs.md`](2026-08-28--audit-report-saved-to-docs.md) — the report-persistence rules this record reuses.
- [`skills/ui-ux-audit/references/rubric.md`](../../skills/ui-ux-audit/references/rubric.md) — Recommendation rule.
- [`skills/ui-ux-audit/references/audit-workflow.md`](../../skills/ui-ux-audit/references/audit-workflow.md) — Phase 2, the bounded browser pass; Phase 5, delivery.
