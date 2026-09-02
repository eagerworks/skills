# `hipaa`'s audit report is a dated file in the audited repo, not terminal output

- **Date:** 2026-08-28

## Context

`pr-review` is the closest existing precedent for a read-only audit skill in this repo, and its
default output is a markdown report printed to the conversation — nothing is written to disk
unless the user explicitly asks for the optional fix loop or a PR comment. `mobile-store-review`
took a different path: it fills `assets/audit-report.md` as its deliverable, but doesn't specify
a fixed location or filename for the completed report, leaving that to the conversation.

`hipaa` needed a firmer answer than either precedent, for a reason specific to this domain: the
audit *is itself potentially discoverable evidence* under HIPAA's own documentation retention
rule. §164.316(b)(2)(i) requires HIPAA-related documentation — which a security assessment like
this audit plainly is — be retained for 6 years from creation or from when it was last in effect.
A report that only ever existed as chat output has no durable existence to retain at all, and a
report saved ad hoc under a name the user chose in the moment (`audit.md`, `notes.md`,
overwritten on the next run) doesn't preserve the trail an org would need if OCR or a customer's
security team later asked "when did you last audit for X, and what did it find."

There's a second, more practical reason: HIPAA audits recur — a codebase changes, PHI-handling
features get added, and re-auditing periodically is expected practice (the same principle
`soc2`'s evidence-and-monitoring reference applies to recurring controls). If every run overwrites
the same file, there's no way to see that drift, or to prove the org has actually been auditing on
a cadence rather than doing it once for a sales conversation and never again.

Two alternatives were considered and rejected:

1. **Terminal output only, like `pr-review`.** Simplest, matches the most common pattern in this
   repo. Rejected because it produces nothing durable — the exact failure mode described above.
2. **A single overwritten file** (e.g. `docs/hipaa-audit.md`, no date in the name). Durable, but
   each new audit destroys the previous one, which is precisely the history the 6-year retention
   expectation assumes exists.

## Decision

The `hipaa` skill always writes its completed report to
`docs/hipaa-audits/YYYY-MM-DD-<slug>.md` in the audited repo, creating the `docs/hipaa-audits/`
directory if it doesn't already exist. This is the **one** file-write action the skill performs by
default — everything else about the audit stays read-only, matching `pr-review`'s default posture
(see `skills/hipaa/references/audit-workflow.md` → "Allowed commands").

The filename is never overwritten. If a report for the same date and slug already exists (e.g. two
audits run the same day with the same scope), the skill appends a numeric suffix rather than
replacing the earlier file.

## Consequences

- A repo using this skill accumulates a dated audit trail over time in `docs/hipaa-audits/`,
  which is itself useful evidence toward the org's own §164.316(b) documentation obligations —
  though the skill doesn't and can't determine on the org's behalf whether that satisfies the
  requirement in full; see `skills/hipaa/references/administrative-and-legal.md`.
- Running the skill against a repo for the first time creates a new top-level directory as a
  side effect, which a user unfamiliar with this convention might not expect from an otherwise
  read-only tool — `SKILL.md` and `references/audit-workflow.md` both call this out explicitly
  so it's never a silent surprise.
- Unlike `pr-review`, there's no option to post the report elsewhere (a PR comment, say) as part
  of this skill's default workflow — the dated file is the canonical output. A user who also wants
  it visible in a PR can paste it manually or ask for that separately; this skill doesn't build it
  in, to keep the default behavior singular and predictable.

## Related

- [2026-06-30--evals-separate-from-skills](2026-06-30--evals-separate-from-skills.md) — the other
  ADR governing where generated/output content lives relative to what ships to users.
- `skills/hipaa/references/audit-workflow.md` → "Phase 6 — Write the report" — the procedure this
  ADR justifies.
- `skills/hipaa/references/audit-controls.md` → "Retention — the 6-year rule" — the underlying
  HIPAA requirement motivating durability in the first place.
- `skills/pr-review/references/workflow.md` — the read-only-by-default precedent this decision
  otherwise follows, with the one deliberate exception explained above.
