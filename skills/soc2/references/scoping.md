# SOC 2 — Scoping Reference

> Covers the decisions that define what the report actually covers: system boundary, Type I vs.
> Type II, observation-period length, subservice organizations, and complementary user entity
> controls. Get these wrong and the whole engagement is either too slow or doesn't satisfy the
> customer asking for it.

## Table of Contents

1. [Defining the system boundary](#defining-the-system-boundary)
2. [Type I vs. Type II](#type-i-vs-type-ii)
3. [Choosing the observation-period length](#choosing-the-observation-period-length)
4. [Subservice organizations](#subservice-organizations)
5. [Complementary user entity controls](#complementary-user-entity-controls)
6. [Multi-product and multi-environment scoping](#multi-product-and-multi-environment-scoping)

---

## Defining the system boundary

The system boundary is the description of infrastructure, software, people, procedures, and data
that the report actually covers. It should be:

- **Narrow enough** that the evidence burden matches what customers actually need reassurance
  about — internal tools unrelated to the customer-facing product usually don't belong.
- **Broad enough** that it includes everything a customer would reasonably assume is "the
  product" — the API, the database, the admin panel used to manage their account, the CI/CD
  pipeline that ships code into it.

A good boundary statement names: the product/service, the infrastructure it runs on (cloud
account(s), regions), the people/teams with access to it, and the major supporting processes
(SDLC, incident response, vendor management). Write it once, early — it drives every other
scoping decision in this file.

---

## Type I vs. Type II

| | Type I | Type II |
|---|---|---|
| Tests | Control **design** at a single point in time | Control design **and operating effectiveness** over a period |
| Typical duration to complete | 6–12 weeks from a standing start | Type I duration + a 3–12 month observation window |
| Evidence | Policies exist, controls are configured correctly *as of the report date* | Everything Type I requires, plus proof each control operated *throughout* the window (logs, tickets, review records, sampling) |
| When it's the right call | Hard deadline inside ~3 months; first-ever report; "prove we're on the path" for a deal that doesn't strictly require Type II | Customer contract explicitly requires it; enough runway to run a real observation window; repeat/ongoing customer relationships that will ask again next year anyway |
| Common trajectory | Often a stepping stone — start the Type II observation window immediately after, so the Type II report is ready ~9–12 months later | — |

**Decision rule:** ask what the customer's contract or security questionnaire literally says.
Many say "SOC 2" without specifying a type — in that case Type I is very often sufficient as a
first step, especially against a deadline. Only commit to Type II from the outset when it's
explicitly required or the organization has the runway to do it once and be done for a year.

---

## Choosing the observation-period length

Once Type II is decided, the period length is a separate choice — typically 3, 6, or 12 months:

- **3 months** — the minimum most auditors will accept for a first Type II. Good when there's
  deal pressure and controls are already operating (not being stood up for the audit).
  Recurring-evidence controls (e.g. a quarterly access review) can only be evidenced once in a
  3-month window, which some auditors flag as thin — confirm with the chosen firm.
  See `references/evidence-and-monitoring.md` for the recurring-control calendar.
- **6 months** — a common middle ground; gives at least one full cycle of monthly controls and
  a defensible sample of quarterly ones.
- **12 months** — the standard for mature, ongoing programs; aligns the report period with a
  fiscal or calendar year and gives the deepest evidence set, at the cost of the longest
  time-to-first-report.

The window **cannot start** until the controls it's meant to evidence are actually operating —
starting it before access reviews, logging, or change-management processes are real just
produces exceptions for the period before they existed. Confirm each in-scope control is live,
then start the clock.

---

## Subservice organizations

A subservice organization is any third party that performs functions relevant to the system
being audited — commonly a cloud hosting provider, payment processor, or data warehouse. Every
one identified in intake needs an explicit decision:

- **Carve-out method** (far more common) — the subservice org's controls are excluded from this
  report; the system description notes that the customer should separately review the subservice
  org's own report (e.g. AWS's own SOC 2, if relying on AWS infrastructure controls). This is why
  most SOC 2 reports don't re-test "does AWS physically secure its data centers" — it's already
  covered by AWS's own report and carved out.
- **Inclusive method** — the subservice org's controls are tested as part of *this* audit. Rare,
  used when the subservice org doesn't have its own attestation and the relationship is critical
  enough that the customer needs assurance anyway.

Default to carve-out for any subservice org that already publishes its own SOC 2 (AWS, GCP,
Azure, most major payment processors) — request their report and reference it, rather than
re-proving what they've already proven.

---

## Complementary user entity controls

Some Common Criteria can only be fully satisfied with cooperation from the *customer's* own
environment — e.g. a customer must configure their own SSO correctly, or manage their own
end-user access to a multi-tenant admin panel. These are documented in the report as
**Complementary User Entity Controls (CUECs)** — controls the customer is responsible for, which
the audited organization's controls assume are in place. Identify these explicitly rather than
letting the audited organization's controls silently assume behavior it can't verify.

---

## Multi-product and multi-environment scoping

- **Multiple products** — each can be its own system boundary with its own report, or bundled
  into one report if they share infrastructure and controls closely enough that testing them
  together is meaningful. Bundling saves audit cost; separating gives cleaner, product-specific
  reports for customers who only care about one.
- **Staging/dev vs. production** — non-production environments are typically excluded from the
  boundary unless customer data flows through them (e.g. production data copied into staging for
  testing — a practice worth flagging as a gap in its own right if discovered during intake).
