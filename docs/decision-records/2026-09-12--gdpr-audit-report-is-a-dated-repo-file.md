# `audit-gdpr`'s audit report is a dated file in the audited repo, not terminal output

- **Date:** 2026-09-12

## Context

This repo now has two competing precedents for where a read-only audit skill's finished report
should live. `pr-review` prints its report to the conversation and writes nothing to disk by
default. `loop-engineering-audit` and `repo-handoff` each write to a single, fixed, overwritten
file (`docs/loop-engineering-audit.md`, `docs/repo-handoff.md`) — durable, but with no history
across runs. `audit-hipaa` took a third path: a dated, never-overwritten file under
`docs/hipaa-audits/`, justified by a HIPAA-specific fact — §164.316(b)(2)(i) requires HIPAA
documentation, which a security assessment like that audit plainly is, be retained for six years
from creation or from when it was last in effect.

`audit-gdpr` needed its own answer rather than defaulting to whichever precedent came first,
because GDPR's own retention rule for *this specific document* doesn't exist — the regulation
never says "keep this audit for N years." What GDPR does provide is arguably a stronger case for
history than HIPAA's fixed retention window:

- **Art 5(2)** makes the controller "responsible for, and able to demonstrate compliance with"
  the Art 5(1) principles — an ongoing burden of proof, not a one-time filing.
- **Art 24(1)** requires implementing measures to "ensure and to be able to demonstrate" compliant
  processing, and does so in a way that implies review over time — the measures have to remain
  appropriate as "the nature, scope, context and purposes of processing" change, which a codebase
  does constantly.
- Unlike HIPAA, GDPR doesn't hand this skill a numeric retention period to point to. The
  justification here rests on the *shape* of the accountability principle rather than a specific
  section, so it's spelled out explicitly rather than asserted by citation alone: demonstrating
  compliance "over time" is only possible if there's a trail showing the org actually re-checked
  itself periodically, rather than doing it once for a sales conversation and never again — the
  same practical argument `audit-hipaa`'s ADR makes, applied to a regulation with a different
  textual basis for it.

Two alternatives were considered and rejected, mirroring `audit-hipaa`'s own ADR:

1. **Terminal output only, like `pr-review`.** Produces nothing durable, so there's no artifact
   left to point to if a customer's security team or a supervisory authority ever asks when the
   codebase was last checked and what it found.
2. **A single overwritten file, like `loop-engineering-audit`/`repo-handoff`** (e.g.
   `docs/gdpr-audit.md`, no date in the name). Durable, but each new audit destroys the record of
   the previous one — precisely the history an Art 5(2)/24(1) accountability argument assumes
   exists, and the trail a supervisory authority inquiry into "were you auditing regularly" would
   need.

## Decision

The `audit-gdpr` skill always writes its completed report to
`docs/gdpr-audits/YYYY-MM-DD-<slug>.md` in the audited repo, creating the `docs/gdpr-audits/`
directory if it doesn't already exist — the same shape as `audit-hipaa`'s
`docs/hipaa-audits/YYYY-MM-DD-<slug>.md`. This is the **one** file-write action the skill performs
by default; everything else stays read-only (see `skills/audit-gdpr/references/audit-workflow.md`
→ "Allowed commands").

The filename is never overwritten. If a report for the same date and slug already exists, the
skill appends a numeric suffix rather than replacing the earlier file — identical behavior to
`audit-hipaa`.

## Consequences

- A repo using this skill accumulates a dated audit trail in `docs/gdpr-audits/` over time, which
  is itself useful evidence toward the org's own Art 5(2)/24(1) accountability obligations —
  though the skill doesn't and can't determine on the org's behalf whether that satisfies the
  obligation in full; see `skills/audit-gdpr/references/accountability-and-legal.md`.
- Running the skill against a repo for the first time creates a new top-level directory as a side
  effect — `SKILL.md` and `references/audit-workflow.md` both call this out explicitly so it's
  never a silent surprise, matching `audit-hipaa`'s own disclosure.
- This repo now has two audit skills on the dated-file pattern (`audit-hipaa`, `audit-gdpr`) and
  two on the single-overwritten-file pattern (`loop-engineering-audit`, `repo-handoff`). That's a
  deliberate split, not drift: the dated-file skills both have a regulatory accountability/
  retention argument specific to the documents they produce; the overwritten-file skills don't
  carry an equivalent argument and instead value a single current source of truth. A future audit
  skill should pick based on whether its own domain has a comparable accountability argument, not
  by copying whichever precedent is closest alphabetically.

## Related

- [2026-08-28--hipaa-audit-report-is-a-dated-repo-file](2026-08-28--hipaa-audit-report-is-a-dated-repo-file.md)
  — the precedent this decision follows, and the ADR whose reasoning this one distinguishes from
  HIPAA's fixed six-year rule.
- [2026-06-30--evals-separate-from-skills](2026-06-30--evals-separate-from-skills.md) — the other
  ADR governing where generated/output content lives relative to what ships to users.
- `skills/audit-gdpr/references/audit-workflow.md` → "Phase 6 — Write the report" — the procedure
  this ADR justifies.
- `skills/audit-gdpr/references/accountability-and-legal.md` → "Accountability — Art 5(2) and
  Art 24" — the underlying GDPR principle motivating durability in the first place.
- `skills/pr-review/references/workflow.md` — the read-only-by-default precedent this decision
  otherwise follows, with the one deliberate exception explained above.
