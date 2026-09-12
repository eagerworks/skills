# Records of Processing Activities (ROPA) — Template

A fillable Art 30 record of processing activities: one row per processing activity, covering
everything Art 30(1) requires a controller to record in writing (Art 30(2) is the shorter
processor-side equivalent, noted below). Build this during Phase 2–5 of the audit
(`references/audit-workflow.md`) from what the audit actually finds — it's a working document the
rest of the audit's findings feed into, not just a final deliverable. Unlike `audit-hipaa`'s
`phi-inventory.md`, this isn't only an audit worksheet: Art 30(1) requires the finished record
exist in writing regardless of whether an audit ever produced one, so a completed version of this
file is itself a piece of the org's compliance evidence.

Note the Art 30(5) exemption: organizations under 250 employees are exempt *unless* the
processing is likely to result in a risk to data subjects, is not occasional, or involves
special-category or criminal-conviction data — most products handling customer data at any real
volume don't actually qualify for this exemption. Confirm applicability with a human before
treating a small org's absent ROPA as low priority.

---

## Controller record — Art 30(1)

> **Example row:**
>
> | Processing activity | (a) Controller contact | (b) Purpose | (c) Data subjects & categories | (d) Recipients | (e) Third-country transfer & safeguard | (f) Retention | (g) Security measures (Art 32(1)) |
> |---|---|---|---|---|---|---|---|
> | Customer support ticketing | privacy@your-org.com | Resolving support requests | Customers — name, email, ticket content | Support platform vendor; LLM summarization vendor | LLM vendor: US-based, SCCs signed, TIA pending | 24 months after ticket closure | TLS in transit; encrypted at rest; role-scoped access |

| Processing activity | (a) Controller contact | (b) Purpose | (c) Data subjects & categories | (d) Recipients | (e) Third-country transfer & safeguard | (f) Retention | (g) Security measures (Art 32(1)) |
|---|---|---|---|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

## Processor record — Art 30(2)

Fill this section instead of (or alongside) the controller record if the audited org is acting as
a processor for some or all of the processing found — see `SKILL.md`'s scoping gate on
controller/processor role. Art 30(2) requires less than Art 30(1): the processor's and each
controller's contact details, the categories of processing carried out on each controller's
behalf, third-country transfer details and safeguards, and a general description of Art 32(1)
security measures — no purpose, data-subject-category, or retention fields, since those are the
controller's determination, not the processor's.

> **Example row:**
>
> | Categories of processing | Controller(s) it's performed for | Third-country transfer & safeguard | Security measures (Art 32(1)) |
> |---|---|---|---|
> | Hosting and serving end-user profile data on behalf of SaaS customers | Each customer using the platform (multi-tenant) | None — all data hosted in `eu-west-1` | Tenant-isolated database roles; encrypted at rest; TLS in transit |

| Categories of processing | Controller(s) it's performed for | Third-country transfer & safeguard | Security measures (Art 32(1)) |
|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

## Notes on scope boundary

Record here any processing activity that was considered but excluded from this ROPA, and why —
e.g. "internal employee payroll data — excluded, out of this audit's scope (HR systems, not the
product codebase)." This keeps the boundary decision visible rather than implicit, matching
`references/personal-data-identification.md`'s guidance on stating ambiguous scope calls
explicitly rather than picking a side silently.

- [PLACEHOLDER]
