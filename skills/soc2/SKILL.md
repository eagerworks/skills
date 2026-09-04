---
name: soc2
description: >
  Expert guide for planning and running a SOC 2 readiness effort — the AICPA attestation report
  that proves an organization's security controls actually operate. Use this skill whenever the
  user: mentions SOC 2, Type I/Type II reports, Trust Services Criteria, or the AICPA; says a
  customer, prospect, or security questionnaire is demanding a SOC 2 report; wants a compliance
  gap analysis or readiness assessment (for their own company or a client's); is choosing between
  Type I and Type II or deciding which Trust Services Categories to include; needs security
  policies (information security, access control, incident response, vendor risk, BCDR, SDLC,
  acceptable use); is setting up access reviews, audit logging, or evidence collection; is
  selecting or preparing for a CPA audit firm; or is evaluating whether to buy a compliance
  platform (Vanta, Drata, Secureframe) versus doing it manually. Also use when the user says
  things like "a customer is asking for our SOC 2", "we need to get compliant to close this
  enterprise deal", or "what security policies do we need?" — even if they don't name SOC 2.
---

# SOC 2 Readiness Skill

SOC 2 is an attestation report issued by a licensed CPA firm, describing an organization's
system and testing whether its security controls were suitably designed (**Type I**, a
point-in-time snapshot) or operated effectively over a period (**Type II**, typically 3–12
months of evidence). It is not a certification and there is no badge to "pass" — the report
itself, exceptions and all, is what customers request. This skill turns a rough starting
situation into two concrete deliverables: a **gap analysis** against the Trust Services
Criteria, and a **phased roadmap** to an audit-ready state.

---

## Step 0 — Frame the Engagement (do this first)

Before asking a single detail question, resolve three things — they change everything
downstream:

1. **Whose SOC 2 is this?** The user's own company, or a client/customer they're assessing on
   behalf of an agency/consultancy? This changes the plan's voice: an internal roadmap the team
   executes themselves, vs. a client-facing gap report plus an engagement plan for a consulting
   relationship. Ask directly if it isn't obvious from context.
2. **Type I or Type II — or undecided?** If undecided, that's fine; the intake below resolves
   it (deadline and existing control maturity drive the answer — see `references/scoping.md`).
3. **What stage is this organization at?**

| Situation | Start here |
|---|---|
| Nothing started — no policies, no formal controls | Full intake below, then Phase 0 of the roadmap |
| Policies exist but were never really followed | Intake below, weight questions toward evidence/operation, not documentation |
| Currently mid-observation window for Type II | Intake below, but read `references/evidence-and-monitoring.md` first — **evidence gaps mid-window cannot be backfilled** |
| Report has exceptions from a prior audit, needs remediation | Intake below, focus on the specific failed criteria, read `references/audit-process.md` on opinion types |

---

## The Intake Interview

**Ask in batches of 3–5 grouped questions — never dump the full list at once.** Wait for
answers, ask sensible follow-ups, and move to the next group. Once the blocking questions
(scope, deadline, infrastructure, existing controls) are answered, stop interviewing and produce
the plan — note anything still unanswered as an explicit **assumption** in the output rather than
stalling on it.

The condensed groups below are enough to produce a first plan. The full bank — with the reason
each question matters and which Trust Services criterion the answer feeds — is in
`references/intake-questionnaire.md`; read it before an engagement that needs real rigor (e.g.
a paid client assessment).

1. **Company & scope** — What does the product do, and what system/product is in scope for the
   report? Team size and how many touch production or customer data? Single product or multiple?
2. **Customers & the driver** — Who's asking, and by when? Is there a specific deal, RFP, or
   security questionnaire behind this? Type I acceptable, or does the customer require Type II?
3. **Infrastructure** — Cloud provider(s)? Managed or self-hosted services? Any subservice
   organizations (payment processor, cloud hosting, data warehouse) that should be carved out vs.
   included in scope?
4. **Identity & access** — SSO/MFA in use? How is access provisioned and, critically,
   *deprovisioned* on offboarding? Is there any periodic access review today?
5. **SDLC & change management** — Version control, PR review, CI, and deploy process? Anyone able
   to push straight to production without review?
6. **Data handling** — What customer/PII data is stored, where, encrypted how (in transit and at
   rest)? Any prior data incidents?
7. **People & posture** — Background checks on hires? Security awareness training? Any prior
   pen test, vulnerability scan, or audit (SOC 2 or otherwise)?
8. **Timeline & budget** — Hard deadline? Budget for a CPA firm and (optionally) a compliance
   platform? Internal owner who will drive this day to day?

---

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| Deciding Type I vs Type II, system boundary, subservice orgs | `references/scoping.md` |
| Understanding the Trust Services Criteria (CC1–CC9 + optional categories) | `references/trust-services-criteria.md` |
| Running a rigorous intake, question-by-question rationale | `references/intake-questionnaire.md` |
| Writing required policies (what auditors actually check) | `references/policies.md` |
| Mapping criteria to concrete engineering controls (SSO, logging, encryption, change mgmt) | `references/technical-controls.md` |
| Collecting evidence without a compliance platform, the recurring-control calendar | `references/evidence-and-monitoring.md` |
| Picking a CPA firm, fieldwork, report anatomy, opinion types, bridge letters | `references/audit-process.md` |
| DIY tooling vs. when a platform (Vanta/Drata/Secureframe) is worth it | `references/tooling.md` |

Copyable templates live in `assets/`:
- `assets/intake-questionnaire.md` — fillable version of the interview for async completion
- `assets/gap-matrix.csv` — spreadsheet-importable control gap tracker
- `assets/readiness-roadmap.md` — phased plan template with a worked example
- `assets/policies/` — nine policy skeletons (infosec, access control, incident response, vendor
  risk, BCDR, SDLC/change management, risk assessment, data classification, acceptable use)

---

## Producing the Plan

Once intake is far enough along, always output in this shape:

1. **Scope statement** — system/product in scope, Type I or II, Trust Services Categories
   included (default to **Security** alone unless the customer specifically requires
   Availability, Confidentiality, Processing Integrity, or Privacy).
2. **Gap matrix** — keyed by Trust Services criterion, each row: requirement, current state,
   status (`Met` / `Partial` / `Missing` / `N/A`), owner. Use `assets/gap-matrix.csv` as the
   shape.
3. **Phased roadmap** — five phases, each with a duration estimate, prerequisites, and exit
   criteria:
   - **Phase 0 — Scope & auditor selection**
   - **Phase 1 — Policies & governance**
   - **Phase 2 — Technical controls**
   - **Phase 3 — Evidence & observation window**
   - **Phase 4 — Fieldwork**
4. **Top 5 — do these first** — the highest-leverage gaps to close immediately.
5. **Assumptions & open questions** — anything not yet answered, stated plainly rather than
   silently guessed.

Realistic calendar: a **Type I** from a standing start typically runs **6–12 weeks**
(policies + controls + a point-in-time review). A **Type II** adds a **3–12 month observation
window** on top of that, because the report period doesn't start until controls are actually
operating — see `references/scoping.md` for how to pick the window length.

---

## Critical Gotchas

1. **SOC 2 is an attestation, not a certification.** There's no badge, seal, or "pass/fail" —
   the CPA firm issues a report describing the system and their test results, exceptions
   included. Never tell a user they'll be "SOC 2 certified."

2. **Only a licensed CPA firm can issue the report.** A compliance platform, consultant, or this
   skill can get an organization *ready* — none of them can perform or sign the audit.

3. **Type II evidence cannot be backfilled.** If a control wasn't operating (or wasn't logged)
   for part of the window, that's a gap for that period — not something a screenshot taken later
   can fix. Start the observation clock only once controls are genuinely running.

4. **Don't scope in categories nobody asked for.**
   ```
   # ✅ correct — matches what the customer's questionnaire actually requires
   Trust Services Categories: Security (Common Criteria only)
   # ❌ wrong — adds months of extra evidence work for categories no one requested
   Trust Services Categories: Security, Availability, Confidentiality, Processing Integrity, Privacy
   ```
   Security (the Common Criteria) alone satisfies the overwhelming majority of vendor security
   reviews. Adding a category means new controls, new evidence, and a longer timeline — only do
   it when a specific customer or contract requires it.

5. **A policy nobody follows is worse than no policy.** Auditors test *operation*, not existence
   — a written access-control policy that's routinely ignored becomes a documented exception.
   Only write down what the team will actually do.

6. **Subservice organizations need a deliberate decision.** Carve-out (excluded, with a note that
   the customer must review the subservice org's own report) vs. inclusive (their controls tested
   as part of this audit) — see `references/scoping.md`. Don't leave it implicit.

7. **Auditor independence.** The firm that designs or remediates controls generally shouldn't be
   the same firm that audits them — check this before recommending a CPA firm that's also doing
   advisory work for the client.

8. **Scope the system boundary narrowly and defensibly.** A boundary that's too broad (e.g. every
   internal tool) multiplies evidence work for no customer benefit; too narrow, and it excludes
   what the customer actually cares about (e.g. the API that processes their data).

9. **A bridge letter covers the gap between report period end and a customer's request date** —
   useful while the next Type II window is still running, but it is not a substitute for the
   report itself.
