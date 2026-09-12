# PHI Inventory — Template

A fillable data-flow inventory: every PHI element, which of the 18 Safe Harbor identifiers it is,
where it's stored, who can read it, which vendors it reaches, and whether a BAA covers each. Build
this during Phase 2 of the audit (`references/audit-workflow.md`) — it's the working document the
rest of the audit is checked against, not just a final deliverable.

---

## Element inventory

> **Example row:**
>
> | Element | Safe Harbor identifier(s) | Stored in | Who can read | Vendors it reaches | BAA confirmed? |
> |---|---|---|---|---|---|
> | `patients.diagnosis_notes` | #18 (unique identifying content) | `patients` table (RDS, encrypted at rest) | Clinicians (role: `clinician`), support staff via shared admin panel | Sentry (via unscrubbed error payloads — see finding), OpenAI (via note-summarizer feature) | Sentry: unknown · OpenAI: unknown |

| Element | Safe Harbor identifier(s) | Stored in | Who can read | Vendors it reaches | BAA confirmed? |
|---|---|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

## Vendor summary

For every distinct vendor named in the table above, one line confirming BAA status — this is the
list `references/administrative-and-legal.md` → "What to hand the user" gets built from.

> **Example:**
> - **Sentry** — BAA status: ⚪ not confirmed. PHI reaches it via unscrubbed error payloads
>   (`config/initializers/sentry.rb` has no `beforeSend` scrub hook). Fix the scrub rule
>   regardless of BAA status; confirm BAA separately if any PHI could still slip through.
> - **AWS RDS** — BAA status: ✅ confirmed (organization-level AWS BAA on file; RDS is on AWS's
>   HIPAA-eligible services list).

- [PLACEHOLDER]

## Notes on scope boundary

Record here any element that was considered but excluded from the PHI inventory, and why — e.g.
"aggregate, de-identified analytics rollups (no per-patient rows) — excluded, not PHI under Safe
Harbor." This keeps the boundary decision visible rather than implicit, matching
`references/phi-identification.md`'s guidance on stating ambiguous calls explicitly rather than
picking a side silently.

- [PLACEHOLDER]
