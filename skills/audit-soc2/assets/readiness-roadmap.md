# SOC 2 Readiness Roadmap — Template

Matches the output format in `SKILL.md` → "Producing the Plan". A worked example is filled in
below each section header so the shape is unambiguous — replace with the real organization's
details.

---

## 1. Scope statement

> **Example:** Your Company, Inc.'s core API and customer dashboard (`app.your.domain.com`),
> hosted on AWS (us-east-1), is in scope for a **Type I** report covering the **Security**
> Trust Services Category (Common Criteria only). AWS infrastructure controls are carved out
> and covered by AWS's own SOC 2 report. Non-production environments are excluded from scope.

- System/product in scope: [PLACEHOLDER]
- Report type: [PLACEHOLDER: Type I / Type II]
- Trust Services Categories: [PLACEHOLDER — default Security only, see `references/trust-services-criteria.md`]
- Subservice organizations and their treatment (carve-out / inclusive): [PLACEHOLDER]

## 2. Gap matrix

See `assets/gap-matrix.csv` for the full spreadsheet-importable version. Summary:

| Status | Count |
|---|---|
| Met | [PLACEHOLDER] |
| Partial | [PLACEHOLDER] |
| Missing | [PLACEHOLDER] |
| N/A | [PLACEHOLDER] |

## 3. Phased roadmap

> **Example durations for a Type I from a standing start (12-person team, single AWS environment):**

| Phase | Focus | Duration | Prerequisites | Exit criteria |
|---|---|---|---|---|
| **Phase 0** | Scope & auditor selection | 1–2 weeks | Intake complete, scope statement agreed | System boundary documented; CPA firm selected and engaged |
| **Phase 1** | Policies & governance | 2–3 weeks | Phase 0 complete | All 9 core policies drafted, owned, and approved (see `assets/policies/`) |
| **Phase 2** | Technical controls | 3–5 weeks | Gap matrix from Phase 0/1 | All "Missing"/"Partial" items in `references/technical-controls.md` closed or scheduled |
| **Phase 3** | Evidence & observation window | Type I: N/A (point-in-time) · Type II: 3–12 months | Phase 2 controls operating | Evidence collected per `references/evidence-and-monitoring.md` for the full window |
| **Phase 4** | Fieldwork | 2–4 weeks | Phase 3 complete | Auditor fieldwork complete; report issued |

- Phase 0: [PLACEHOLDER]
- Phase 1: [PLACEHOLDER]
- Phase 2: [PLACEHOLDER]
- Phase 3: [PLACEHOLDER]
- Phase 4: [PLACEHOLDER]

## 4. Top 5 — do these first

> **Example:**
> 1. Enforce MFA org-wide for anything touching production or customer data.
> 2. Enable required PR review (branch protection) on every production repo.
> 3. Tie access revocation to the HR offboarding process.
> 4. Adopt and approve the Information Security and Access Control policies.
> 5. Schedule the annual penetration test now — lead times run several weeks.

1. [PLACEHOLDER]
2. [PLACEHOLDER]
3. [PLACEHOLDER]
4. [PLACEHOLDER]
5. [PLACEHOLDER]

## 5. Assumptions & open questions

> **Example:** Assumed at-rest encryption is enabled on all RDS/S3 resources — not yet verified;
> load-bearing assumption, confirm before Phase 2 sign-off.

- [PLACEHOLDER]
