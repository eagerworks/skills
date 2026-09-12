# SOC 2 Readiness — Intake Questionnaire

A fillable version of the interview for async completion by a client or internal stakeholder.
Answer what you can; leave the rest blank and note "not sure" rather than guessing — unanswered
items become explicit assumptions in the readiness plan rather than silent gaps.

## 1. Company & scope

- What does the product do, in one or two sentences?
- What specific system/product should be in scope for the report?
- How many people are on the team, and how many can touch production or customer data?
- Is this a single product/environment, or multiple (e.g. staging + prod, multiple apps)?

## 2. Customers & the driver

- Who is asking for this, and by when?
- Is there a specific deal, RFP, or security questionnaire behind this? (attach it if so)
- Does the customer explicitly require Type II, or would Type I satisfy them?
- Which Trust Services Categories does the requirement actually name (if any)?

## 3. Infrastructure

- Which cloud provider(s), and which services are in scope?
- Anything self-hosted or on infrastructure the team directly manages?
- Any subservice organizations (payment processor, cloud hosting, data warehouse, email provider)
  handling customer data?
- Is there already logging/monitoring in place, and where does it go?

## 4. Identity & access

- Is SSO in place, and is MFA enforced for anything touching production or customer data?
- How is access granted when someone joins or changes roles — and revoked when they leave?
- Is there any periodic access review today?
- Who has standing admin/root access to production, cloud consoles, or the database?

## 5. SDLC & change management

- Where's the code — what version control, and is PR review required before merge?
- Is there CI, and does anything gate a deploy?
- Can anyone push directly to production without going through the pipeline?
- How are infrastructure changes made — console clickops, or infrastructure-as-code with review?

## 6. Data handling

- What customer or personal data does the system store, and where?
- Is data encrypted in transit and at rest — and how?
- Has there ever been a data incident, breach, or unauthorized access event?
- Is there a data retention and deletion policy, followed in practice?

## 7. People & posture

- Are background checks run on new hires touching production or customer data?
- Is there security awareness training, and how often?
- Has the organization had any prior audit — SOC 2 or otherwise — including exceptions or
  findings?
- Has there been a penetration test or vulnerability scan in the last 12 months?

## 8. Timeline & budget

- Is there a hard deadline, and what's driving it?
- What's the budget for a CPA firm and (if relevant) a compliance platform?
- Is there a named internal owner who will drive this day to day?

---

See `references/intake-questionnaire.md` in the skill for the rationale behind each question and
which Trust Services criterion it feeds.
