# SOC 2 — Policies Reference

> Covers the required policy set, what each must actually contain to survive an auditor, review
> cadence, and common objections. Skeletons for every policy listed here live in
> `assets/policies/` — this file is the "why" and "what good looks like"; the assets are the
> starting document.

## Table of Contents

1. [The required policy set](#the-required-policy-set)
2. [What makes a policy audit-ready](#what-makes-a-policy-audit-ready)
3. [Review cadence and versioning](#review-cadence-and-versioning)
4. [Common auditor objections](#common-auditor-objections)

---

## The required policy set

Nine policies cover the large majority of what a Security-only (Common Criteria) SOC 2 report
needs. Each maps primarily to one or two Common Criteria series, though most touch CC1/CC2
(governance and communication) as well.

| Policy | Primary criteria | Asset |
|---|---|---|
| Information Security Policy | CC1, CC2 (the umbrella policy — often referenced by all the others) | `assets/policies/information-security-policy.md` |
| Access Control Policy | CC6 | `assets/policies/access-control-policy.md` |
| Incident Response Plan | CC7 | `assets/policies/incident-response-plan.md` |
| Vendor Risk Management Policy | CC9 | `assets/policies/vendor-risk-management-policy.md` |
| Business Continuity & Disaster Recovery Plan | CC9 | `assets/policies/business-continuity-and-dr-plan.md` |
| SDLC & Change Management Policy | CC8 | `assets/policies/sdlc-and-change-management-policy.md` |
| Risk Assessment Policy | CC3 | `assets/policies/risk-assessment-policy.md` |
| Data Classification & Retention Policy | CC6, CC9 | `assets/policies/data-classification-and-retention-policy.md` |
| Acceptable Use Policy | CC1, CC6 | `assets/policies/acceptable-use-policy.md` |

If an optional Trust Services Category is added (see `trust-services-criteria.md`), it usually
brings its own additional policy needs — e.g. Privacy needs a privacy notice and a data subject
rights procedure, which isn't covered by the nine above.

---

## What makes a policy audit-ready

A policy survives audit scrutiny when it has all of the following — a document missing any of
these reads as informal, and an auditor will ask about it:

1. **An owner** — a named role (not "the team") accountable for the policy's content and
   enforcement.
2. **An approval record** — who approved it, and when. Even a lightweight sign-off (a merged PR,
   a dated approval in a doc tool) counts, as long as it's real and timestamped.
3. **A review cadence, and evidence the review actually happened** — annual review is standard;
   the evidence is a dated note or version-history entry, not just "we'd review it if needed."
4. **Content that matches what the organization actually does** — this is the part most often
   skipped. A policy copied from a template that says things the organization doesn't follow
   creates a documented exception the moment an auditor samples it.
5. **Acknowledgement, where relevant** — policies that govern employee behavior (acceptable use,
   information security) should have a record that employees have read and agreed to them,
   typically at onboarding and annually thereafter.

---

## Review cadence and versioning

- **Annual review minimum** for every policy in the set, even if nothing changes — the review
  itself, and its date, is what an auditor is looking for.
- **Version history** — keep policies in version control (a repo, or a doc tool with visible
  revision history) so "when was this last changed and by whom" has a real answer instead of
  relying on memory.
- **Re-review triggers** beyond the annual cycle: a relevant incident, a significant
  infrastructure or vendor change, or a prior audit exception tied to that policy.

---

## Common auditor objections

- **"This policy describes a process that doesn't match what we saw in evidence."** The single
  most common finding. Fix by writing policies to match actual practice, not aspirational
  practice — if the org doesn't really do quarterly access reviews yet, don't write a policy
  that says it does until that's true.
- **"There's no record this was approved or reviewed."** Fix with the lightweight approval record
  described above — it doesn't need to be elaborate, just real and dated.
- **"Employees can't demonstrate awareness of this policy."** Fix with an acknowledgement record
  at onboarding and on each annual update.
- **"This policy is generic and doesn't reflect our actual environment."** A template with
  placeholder text never replaced (cloud provider names, tool names, specific procedures) is an
  immediate tell. Every asset in `assets/policies/` has explicit `[PLACEHOLDER]` markers for this
  reason — replace every one before treating a policy as finished.
