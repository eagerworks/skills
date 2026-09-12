# HIPAA Audit Report — Template

Matches the output format in `SKILL.md` → "The Audit in Six Phases" (Phase 6) and
`references/audit-workflow.md`. Write the filled version to
`docs/hipaa-audits/YYYY-MM-DD-<slug>.md` in the audited repo — never overwrite an existing dated
report. A worked example is shown below each section header; replace with the real findings.

---

## Audit metadata

> **Example:**
> - **Repo:** your-org/patient-portal
> - **Date:** 2026-08-28
> - **Scope:** Full technical safeguards audit (§164.312)
> - **Role classification:** Business associate (customer is a covered entity clinic network)
> - **Reviewer:** Claude, via the `hipaa` skill

- Repo: [PLACEHOLDER]
- Date: [PLACEHOLDER]
- Scope: [PLACEHOLDER]
- Role classification: [PLACEHOLDER — covered entity / business associate / subcontractor / neither]
- Reviewer: [PLACEHOLDER]

## 🔴 Blockers

> **Example:**
> - **`app/models/patient.rb:14`** — `logger.info("Updated: #{patient.inspect}")` writes full
>   patient record (name, DOB, diagnosis) to plaintext application logs.
>   - **Safeguard:** §164.312(e) transmission/storage of PHI without protection; also
>     `references/phi-in-code.md` → Application logs.
>   - **Fix:** log `patient.id` only; add PHI fields to `filter_parameters`.

- [PLACEHOLDER]

## 🟡 Risks

> **Example:**
> - **`config/initializers/sentry.rb`** — Sentry scrub list covers `password` and `ssn` but not
>   `diagnosis_notes`, a field added after the scrub list was last updated.
>   - **Safeguard:** §164.312(e); `references/phi-in-code.md` → Exception trackers.
>   - **Fix:** add `diagnosis_notes` and audit the scrub list against the current schema.

- [PLACEHOLDER]

## ⚪ Needs a Human

> **Example:**
> - **BAA with [LLM vendor]** — `app/services/note_summarizer.rb:22` sends clinical notes to a
>   model API. No BAA status confirmable from code.
>   - **Safeguard:** `references/administrative-and-legal.md` → Business Associate Agreements.
>   - **Action:** confirm a signed BAA covers this specific vendor and workload before this flow
>     continues in production.

- [PLACEHOLDER]

## 🟢 Pass Summary

> **Example:** MFA enforced org-wide via SSO provider (`references/technical-safeguards.md`
> §164.312(d)); RDS storage encryption confirmed active on all instances
> (`references/infrastructure.md`); PR review required on all production repos (integrity control,
> §164.312(c)).

- [PLACEHOLDER]

## Assumptions & Unverified

> **Example:** Assumed the `analytics-events` package (a workspace dependency) doesn't forward
> raw request bodies to Segment — not confirmed from its source, which lives in a private
> registry this audit didn't have access to. Verify before Phase 3 sign-off.

- [PLACEHOLDER]
