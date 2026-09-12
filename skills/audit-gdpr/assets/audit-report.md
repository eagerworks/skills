# GDPR Audit Report — Template

Matches the output format in `SKILL.md` → "The Audit in Six Phases" (Phase 6) and
`references/audit-workflow.md`. Write the filled version to
`docs/gdpr-audits/YYYY-MM-DD-<slug>.md` in the audited repo — never overwrite an existing dated
report. A worked example is shown below each section header; replace with the real findings.

---

## Audit metadata

> **Example:**
> - **Repo:** your-org/booking-platform
> - **Date:** 2026-09-12
> - **Scope:** Full audit — data subject rights and outbound processor review
> - **Role classification:** Processor (customer is the controller for its own end users);
>   Art 3(2)(a) targeting applies via EU-priced plans
> - **Reviewer:** Claude, via the `audit-gdpr` skill

- Repo: [PLACEHOLDER]
- Date: [PLACEHOLDER]
- Scope: [PLACEHOLDER]
- Role classification: [PLACEHOLDER — controller / processor / joint controller, and the Art 3 basis that applies]
- Reviewer: [PLACEHOLDER]

## 🔴 Blockers

> **Example:**
> - **`app/models/user.rb:erase!`** — `update!(deleted_at: Time.current)` leaves name, email,
>   and address fully intact; the row is only hidden from default query scope.
>   - **Obligation:** Art 17 right to erasure; also `references/data-subject-rights.md` →
>     "Art 17 — right to erasure."
>   - **Fix:** scrub personal-data fields in place (not just set a flag), and extend the same
>     transaction to the search index and cache — see the same reference's code example.

- [PLACEHOLDER]

## 🟡 Risks

> **Example:**
> - **`config/initializers/sentry.rb`** — no `before_send` scrub configured; default capture
>   includes full request bodies on the signup and checkout endpoints.
>   - **Obligation:** Art 32(1) security of processing; `references/personal-data-in-code.md` →
>     "Exception trackers."
>   - **Fix:** add a scrub hook excluding known personal-data fields before events are sent.

- [PLACEHOLDER]

## ⚪ Needs a Human

> **Example:**
> - **Processor contract with [LLM vendor]** — `app/services/summarizer.rb:18` sends support
>   ticket bodies, including customer names, to a model API. No Art 28(3) contract status
>   confirmable from code.
>   - **Obligation:** `references/transfers-and-processors.md` → "Processor contracts — Art 28."
>   - **Action:** confirm a signed Data Processing Agreement covers this vendor and this specific
>     workload before this flow continues in production.

- [PLACEHOLDER]

## 🟢 Pass Summary

> **Example:** TLS enforced on all personal-data-bearing endpoints (Art 32(1)(a));
> cookie banner blocks analytics/ad scripts until explicit consent is given
> (`references/lawful-basis-and-consent.md` → ePrivacy interaction); access export endpoint
> includes the full Art 15(1) metadata block, not just raw data rows.

- [PLACEHOLDER]

## Assumptions & Unverified

> **Example:** Assumed the `us-east-1` region hosts all primary data based on
> `config/database.yml`; did not confirm no read replica or backup exists in another region.
> Verify before sign-off.

- [PLACEHOLDER]
