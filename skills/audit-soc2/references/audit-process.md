# SOC 2 — Audit Process Reference

> Covers selecting a CPA firm, readiness assessment vs. the audit itself, fieldwork mechanics,
> report anatomy, opinion types, bridge letters, and realistic cost/timeline ranges.

## Table of Contents

1. [Selecting a CPA firm](#selecting-a-cpa-firm)
2. [Readiness assessment vs. the audit](#readiness-assessment-vs-the-audit)
3. [Fieldwork mechanics and the PBC list](#fieldwork-mechanics-and-the-pbc-list)
4. [Report anatomy](#report-anatomy)
5. [Opinion types and what an exception means](#opinion-types-and-what-an-exception-means)
6. [Bridge letters](#bridge-letters)
7. [Realistic cost and timeline ranges](#realistic-cost-and-timeline-ranges)

---

## Selecting a CPA firm

Only a licensed CPA firm (or a firm with licensed CPAs on staff performing the attestation
engagement) can issue a SOC 2 report — this rules out compliance platforms and consultants as
the actual auditor, however much readiness work they do.

What to ask a candidate firm:

- **Independence** — do they also provide advisory/remediation services to this organization?
  If a firm designed the controls it's about to audit, that's a conflict most reputable firms
  will decline on their own, but confirm it explicitly.
- **Relevant experience** — have they audited companies of similar size/stack/industry before?
  A firm unfamiliar with, say, a modern cloud-native stack may ask for evidence in ways that
  don't map cleanly to how the org actually operates.
- **Turnaround and availability** — when can fieldwork realistically start, and how long does
  report issuance take after fieldwork ends (commonly 2–6 weeks)?
- **Fixed fee vs. hourly**, and what's included — a readiness assessment, a Type I as a stepping
  stone to Type II, ongoing annual audits.

## Readiness assessment vs. the audit

Many firms (and independent consultants) offer a **readiness assessment** — a review against the
Trust Services Criteria before the formal audit, surfacing gaps while there's still time to fix
them. This is optional but valuable, especially for a first-ever SOC 2, and is exactly the kind
of gap analysis this skill's plan output is designed to produce or corroborate. The formal audit
itself follows only once readiness work has closed the major gaps.

## Fieldwork mechanics and the PBC list

Once ready, the auditor issues a **PBC list** ("provided by client") — the specific artifacts
they need: policy documents, access exports, sampled tickets, scan results, and so on. Fieldwork
is the period where the auditor reviews this evidence, asks follow-up questions, and may sample
additional items not on the original list. Having evidence organized per `evidence-and-monitoring.md`
before the PBC list arrives turns fieldwork from a scramble into mostly a matter of handing over
files that already exist.

## Report anatomy

A SOC 2 report has four main sections:

1. **Management's assertion** — the organization's own statement describing the system and
   asserting its controls are suitably designed (Type I) or designed and operating effectively
   (Type II).
2. **System description** — a detailed narrative of the system boundary, infrastructure, and
   controls, prepared by the organization and reviewed by the auditor.
3. **The auditor's opinion** — the CPA firm's independent conclusion (see opinion types below).
4. **Description of tests and results** (Type II only) — for each control, what the auditor
   tested, the sample examined, and the result — including any exceptions found.

## Opinion types and what an exception means

- **Unqualified opinion** — controls were suitably designed (and, for Type II, operating
  effectively) with no material exceptions. The outcome every organization is aiming for.
- **Qualified opinion** — one or more controls had exceptions, but they don't undermine the
  report as a whole; the specific exceptions are disclosed in detail. This is not automatically
  disqualifying to a customer — many will accept a qualified report with a documented remediation
  plan, especially for a first Type II.
- **Adverse opinion** — the exceptions are pervasive enough that the controls, taken as a whole,
  don't achieve their objective. Rare, and usually means the org went to audit before it was
  ready.

An **exception** noted in fieldwork isn't necessarily fatal — it's a factual finding ("in 2 of 25
sampled terminations, deprovisioning took longer than the documented SLA") that gets disclosed
rather than hidden. The right response is remediation before the next reporting period, not
panic about the current one.

## Bridge letters

A bridge letter (also called a gap letter) is a short attestation from the organization covering
the period between the end of the last report and the date a customer needs assurance —
e.g. the Type II report covers through March, a customer needs assurance in May. It states that,
to the organization's knowledge, no material changes occurred to the control environment in the
gap period. It's a stopgap, not a substitute for keeping the next report's window running
continuously — customers generally accept one bridge letter, not an indefinite stream of them.

## Realistic cost and timeline ranges

Figures vary widely by firm, org size, and scope — use these as planning ranges to set
expectations, not quotes:

| Item | Typical range |
|---|---|
| CPA firm fee, Type I | Low-to-mid five figures (varies heavily by firm and org size) |
| CPA firm fee, Type II | Higher than Type I — more testing, more fieldwork time |
| Readiness assessment (if separate from the audit) | Low five figures, or bundled into the audit engagement |
| Time to Type I from a standing start | 6–12 weeks |
| Time to Type II from a standing start | Type I timeline + a 3–12 month observation window |
| Time from end of fieldwork to report issuance | 2–6 weeks |

Always confirm current figures with candidate firms directly — this table is for setting
realistic roadmap expectations during planning, not for quoting a client.
