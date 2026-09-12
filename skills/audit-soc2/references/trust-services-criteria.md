# SOC 2 — Trust Services Criteria Reference

> Covers the AICPA Trust Services Criteria (2017, as revised): the Common Criteria (Security)
> that every SOC 2 report includes, and the four optional categories. Use this to translate a
> customer's ask or an intake answer into the specific criteria a plan must address.

## Table of Contents

1. [How the criteria are structured](#how-the-criteria-are-structured)
2. [Common Criteria (Security) — CC1 through CC9](#common-criteria-security--cc1-through-cc9)
3. [The four optional categories](#the-four-optional-categories)
4. [Deciding which categories to include](#deciding-which-categories-to-include)
5. [From criteria to controls](#from-criteria-to-controls)

---

## How the criteria are structured

Every SOC 2 report is built on the **Trust Services Criteria (TSC)**, organized into five
categories:

| Category | Required? |
|---|---|
| **Security** (the "Common Criteria", CC1–CC9) | Always — every SOC 2 report includes this |
| Availability | Optional |
| Confidentiality | Optional |
| Processing Integrity | Optional |
| Privacy | Optional |

Within each category, criteria break down further into **points of focus** — illustrative,
non-exhaustive descriptions of what a control satisfying that criterion might look like. Points
of focus are guidance, not a checklist to satisfy item-by-item; an organization can meet a
criterion in a way that doesn't map to every point of focus listed for it.

---

## Common Criteria (Security) — CC1 through CC9

Every SOC 2 report — Type I or Type II, whatever optional categories are added — includes these
nine series. This is the backbone the gap matrix should be organized around.

| Series | Name | What it's really asking |
|---|---|---|
| **CC1** | Control Environment | Does leadership actually value security — board oversight, org structure, accountability, competence of people in security-relevant roles? |
| **CC2** | Communication & Information | Do people know the policies exist and understand their responsibilities? Is security communicated internally and to relevant external parties? |
| **CC3** | Risk Assessment | Is there a documented process for identifying and assessing risks to the system, including fraud risk and risk from change? |
| **CC4** | Monitoring Activities | Are controls actually being checked over time — internal audits, control self-assessments, management review — not just designed once and forgotten? |
| **CC5** | Control Activities | Are the specific technical and process controls that mitigate identified risks actually implemented (this is where most technical work lands)? |
| **CC6** | Logical & Physical Access Controls | Authentication, authorization, provisioning/deprovisioning, encryption, physical security of infrastructure — the criterion auditors spend the most time testing. |
| **CC7** | System Operations | Vulnerability management, monitoring/alerting/detection, incident response, capacity planning. |
| **CC8** | Change Management | Is there a controlled process for authorizing, testing, and approving changes before they reach production? |
| **CC9** | Risk Mitigation | Vendor/subservice organization risk management and business continuity/disaster recovery planning. |

**Where the evidence actually gets hard:** CC6 (access) and CC7 (operations) generate the bulk
of Type II evidence requests — access review artifacts, deprovisioning tickets, vulnerability
scan results, alert/incident logs — because they're the criteria that require proof of ongoing
operation, not a one-time document.

---

## The four optional categories

Add a category only when a specific customer contract, RFP, or regulatory driver requires it —
see [Deciding which categories to include](#deciding-which-categories-to-include). Each one adds
real, ongoing evidence burden.

### Availability

Whether the system is available for operation and use as committed (uptime SLAs, capacity
planning, incident response for outages, backup and recovery testing). Relevant if customers
have contractual uptime commitments that flow through to the report.

### Confidentiality

Whether information designated as confidential is protected as committed (contractual
non-disclosure obligations beyond standard PII — e.g. a customer's proprietary business data,
source code, or trade secrets held on their behalf).

### Processing Integrity

Whether system processing is complete, valid, accurate, timely, and authorized. Relevant for
systems that compute something customers rely on being correct — billing calculations, financial
transaction processing, data pipelines feeding a customer's own reporting.

### Privacy

Whether personal information is collected, used, retained, disclosed, and disposed of in
conformity with the entity's privacy notice and generally accepted privacy principles. This is
the heaviest of the four to add — it requires a privacy notice, consent mechanisms, data subject
rights handling, and often overlaps with (but does not replace) GDPR/CCPA compliance work.

---

## Deciding which categories to include

Default to **Security only**. It satisfies the large majority of vendor security reviews,
because most enterprise procurement teams are really asking "can we trust your access controls,
your monitoring, and your incident response" — which is exactly what the Common Criteria covers.

Add a category when, and only when:

- A specific customer contract or RFP explicitly requires it (read the actual language — "SOC 2
  Type II" in a contract almost always means Security only unless another category is named).
- There's a regulatory or contractual commitment that naturally maps to it (e.g. an uptime SLA →
  Availability; a data processing agreement with strict retention/disposal language → Privacy).
- The product's core value proposition depends on it being independently verified (e.g. a
  billing/payments product might proactively add Processing Integrity as a sales differentiator).

Each additional category multiplies the evidence and control work, and extends the Type II
observation window's evidence burden proportionally — this is a scope decision with real cost,
not a box to tick for completeness.

---

## From criteria to controls

The TSC describes *what* must be true, not *how* to achieve it — that mapping to concrete
engineering and process work is in `references/technical-controls.md` (for CC5–CC8, the
technical bulk) and `references/policies.md` (for CC1–CC3, the governance bulk). Use this file
to identify which criteria are in play; use those two for what to actually go build.
