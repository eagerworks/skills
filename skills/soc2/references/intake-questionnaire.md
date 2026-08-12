# SOC 2 — Full Intake Questionnaire Reference

> The compact version in `SKILL.md` is enough for a first-pass plan. Read this file before an
> engagement that needs real rigor — a paid client assessment, or any situation where getting an
> answer wrong sends the roadmap in the wrong direction. Ask in the same batched groups; this
> file adds the *why* behind each question and what it changes in the output.

## Table of Contents

1. [How to run the interview](#how-to-run-the-interview)
2. [Group 1 — Company & scope](#group-1--company--scope)
3. [Group 2 — Customers & the driver](#group-2--customers--the-driver)
4. [Group 3 — Infrastructure](#group-3--infrastructure)
5. [Group 4 — Identity & access](#group-4--identity--access)
6. [Group 5 — SDLC & change management](#group-5--sdlc--change-management)
7. [Group 6 — Data handling](#group-6--data-handling)
8. [Group 7 — People & posture](#group-7--people--posture)
9. [Group 8 — Timeline & budget](#group-8--timeline--budget)
10. [When an answer is unavailable](#when-an-answer-is-unavailable)

---

## How to run the interview

Ask 3–5 questions per group, in order — later groups build on earlier answers (e.g. the system
boundary from Group 1 determines which infrastructure in Group 3 is actually in scope). Skip a
question if the answer is already evident from context (a user who's pasted their `deploy.yml`
has already answered "cloud provider"). Once scope, deadline, infrastructure, and existing
control maturity are known, stop interviewing — produce the plan and list anything unanswered as
an assumption.

---

## Group 1 — Company & scope

| Question | Why it's asked | Feeds |
|---|---|---|
| What does the product do, in one or two sentences? | Establishes what data flows through the system and what customers are trusting it with. | System description (used verbatim in the report) |
| What specific system/product is in scope for the report? | The system boundary is the single most consequential scoping decision — too broad multiplies evidence work, too narrow excludes what the customer cares about. | CC1, system boundary |
| How many people are on the team, and how many can touch production or customer data? | Small teams (<15) often can satisfy access controls with simpler tooling (native cloud IAM) than larger ones need. | CC6 control design |
| Is this a single product/environment, or multiple (e.g. staging + prod, multiple apps)? | Multiple systems may need their own scoping decision or a documented boundary excluding non-production. | System boundary |

## Group 2 — Customers & the driver

| Question | Why it's asked | Feeds |
|---|---|---|
| Who is asking for this, and by when? | The real deadline — often an enterprise deal's close date — is what actually drives Type I vs. Type II and the roadmap pace. | Timeline, Type I/II decision |
| Is there a specific deal, RFP, or security questionnaire behind this? | If a questionnaire already lists required controls, it's a ready-made partial gap matrix. Ask to see it. | Gap matrix seed |
| Does the customer explicitly require Type II, or would Type I (or even a completed questionnaire) satisfy them? | Many procurement teams say "SOC 2" without knowing they'd accept Type I as an interim step — this alone can cut the timeline from a year to two months. | Type I/II decision |
| Which Trust Services Categories does the requirement actually name? | Read the literal contract/RFP language — don't default to more than Security without a specific reason (see `trust-services-criteria.md`). | Scope statement |

## Group 3 — Infrastructure

| Question | Why it's asked | Feeds |
|---|---|---|
| Which cloud provider(s), and which services are in scope? | Determines which native controls (IAM, CloudTrail/Cloud Audit Logs, GuardDuty, Security Command Center, etc.) are available for free vs. need to be built. | CC6, CC7 technical controls |
| Anything self-hosted or on infrastructure the team directly manages? | Self-managed infra needs its own patching, hardening, and monitoring story — can't lean on a cloud provider's shared-responsibility controls. | CC7 |
| Any subservice organizations — payment processor, cloud hosting, data warehouse, email provider handling customer data? | Each one needs an explicit carve-out (excluded, customer reviews their own report) or inclusive (tested as part of this audit) decision. | Scoping — see `scoping.md` |
| Is there already logging/monitoring in place, and where does it go? | Determines whether CC7 evidence (detection, alerting) is a build from scratch or already flowing somewhere. | CC7 |

## Group 4 — Identity & access

| Question | Why it's asked | Feeds |
|---|---|---|
| Is SSO in place, and is MFA enforced for anything touching production or customer data? | The single highest-value control for CC6 — usually the first thing auditors ask about. | CC6 |
| How is access granted when someone joins or changes roles — and, critically, how is it revoked when they leave? | Deprovisioning failures are the most common Type II exception. An answer of "we remember to do it" is a gap. | CC6 |
| Is there any periodic access review today (who has access to what, confirmed still appropriate)? | Quarterly access reviews are a near-universal Type II evidence requirement — if none exist, this becomes a Phase 1/2 item immediately. | CC6, recurring evidence |
| Who has standing admin/root access to production, cloud consoles, or the database? | A large or undocumented admin list is a common finding — the plan should recommend narrowing it. | CC6 |

## Group 5 — SDLC & change management

| Question | Why it's asked | Feeds |
|---|---|---|
| Where's the code — what version control, and is PR review required before merge? | CC8 requires evidence that changes are reviewed before reaching production; an org with no required review has a gap here regardless of how good the code is. | CC8 |
| Is there CI, and does anything gate a deploy (tests, required checks)? | Feeds the change-management control narrative and gives ready-made evidence (CI logs) if it already exists. | CC8 |
| Can anyone push directly to production without going through the pipeline? | A "yes" here is one of the most common Type II exceptions — direct production access bypassing change control. | CC8 |
| How are infrastructure changes made — console clickops, or infrastructure-as-code with review? | IaC with PR review is much easier to evidence than console changes, which leave no reviewable trail. | CC8 |

## Group 6 — Data handling

| Question | Why it's asked | Feeds |
|---|---|---|
| What customer or personal data does the system store, and where? | Determines encryption and retention requirements, and whether Privacy/Confidentiality categories are worth discussing. | CC6, optional categories |
| Is data encrypted in transit and at rest — and if so, how (provider-managed keys, customer-managed, application-level)? | A "we assume the cloud provider handles it" answer needs verification, not just an assumption, before it goes in the gap matrix as "Met." | CC6 |
| Has there ever been a data incident, breach, or unauthorized access event? | Directly relevant to CC3 (risk assessment) and, if unresolved, may need remediation before the report can be clean. | CC3, CC7 |
| Is there a data retention and deletion policy, followed in practice? | A written policy with no enforcement is the classic "policy nobody follows" trap (see SKILL.md gotcha #5). | CC1, CC9 |

## Group 7 — People & posture

| Question | Why it's asked | Feeds |
|---|---|---|
| Are background checks run on new hires, at least for roles touching production or customer data? | A common CC1 control; if missing, cheap and fast to add going forward (can't retroactively background-check existing staff, but the policy going forward still counts). | CC1 |
| Is there security awareness training, and how often? | Needs to be recurring (typically annual) with tracked completion — a one-time onboarding mention doesn't count as evidence. | CC1, recurring evidence |
| Has the organization had any prior audit — SOC 2 or otherwise — including any exceptions or findings? | Prior exceptions are the fastest way to know exactly what an auditor will scrutinize this time. | Gap matrix seed |
| Has there been a penetration test or vulnerability scan in the last 12 months? | Often a near-universal Type II evidence requirement (annual pen test); if none exists, it needs scheduling early — third-party pen testers can have multi-week lead times. | CC7 |

## Group 8 — Timeline & budget

| Question | Why it's asked | Feeds |
|---|---|---|
| Is there a hard deadline, and what's driving it? | Sets the pace of the whole roadmap — a 6-week deadline forces Type I; a 9-month runway makes Type II realistic. | Type I/II decision, roadmap pacing |
| What's the budget for a CPA firm, and (if relevant) a compliance platform? | CPA fees and platform subscriptions vary widely — see `audit-process.md` and `tooling.md` for realistic ranges to set expectations early. | Phase 0 |
| Is there a named internal owner who will drive this day to day? | Readiness efforts without a single accountable owner stall — this is worth surfacing as a risk if the answer is "nobody specific." | Roadmap ownership |

---

## When an answer is unavailable

Don't block the plan on a missing answer. Instead:

- State the assumption explicitly in the plan's **Assumptions & open questions** section.
- Default conservatively — e.g. assume no MFA rather than assuming it exists, since the gap
  matrix should surface risk, not hide it.
- Flag which assumptions are load-bearing (change the Type I/II recommendation or the timeline)
  vs. cosmetic (only affect wording), so the user knows which ones to confirm before acting on
  the plan.
