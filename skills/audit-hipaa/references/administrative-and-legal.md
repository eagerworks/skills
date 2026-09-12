# HIPAA — Administrative & Legal (Needs a Human)

> Not legal advice. This file exists to name what a static code audit cannot verify, so those
> items get routed to a human rather than silently skipped or guessed at. Every item below is
> graded ⚪ **Needs a human** in the audit report — never 🟢, regardless of how confident a
> document in the repo looks (see `SKILL.md` Gotcha 6).

## Table of Contents

1. [Why this is out of scope for a code audit](#why-this-is-out-of-scope-for-a-code-audit)
2. [Business Associate Agreements](#business-associate-agreements)
3. [Risk analysis](#risk-analysis)
4. [Workforce training and sanctions](#workforce-training-and-sanctions)
5. [Breach notification](#breach-notification)
6. [Minimum necessary](#minimum-necessary)
7. [Patient right of access](#patient-right-of-access)
8. [What to hand the user](#what-to-hand-the-user)

---

## Why this is out of scope for a code audit

The HIPAA Security Rule has three safeguard categories — administrative (§164.308), physical
(§164.310), and technical (§164.312). This skill's focus is technical safeguards, because that's
what's visible in source code. Administrative and physical safeguards are organizational and
legal in nature — a repo can contain a *policy document about* them, but code review cannot
confirm the policy is actually followed, that a physical facility is secured, or that a legal
agreement is validly executed. Treat a policy doc's presence as evidence a process is *documented*,
never as evidence it's *operating* — same discipline `soc2` applies to its own policies.

## Business Associate Agreements

A BAA is a legally binding contract between a covered entity and a business associate (or between
a business associate and its subcontractor) governing how PHI is handled. Code can show that PHI
flows to a vendor (`references/phi-in-code.md`, `references/infrastructure.md`); it cannot confirm
a BAA is signed, current, or actually covers the specific service/workload in use. When PHI flows
to a vendor with no confirmation a BAA is in place, grade the *flow itself* 🔴 (per `SKILL.md`
Gotcha 3) and separately note the BAA-confirmation task as ⚪ for a human to close out.

## Risk analysis

§164.308(a)(1)(ii)(A) requires a periodic, documented risk analysis assessing potential risks to
ePHI confidentiality, integrity, and availability. This is a formal organizational process, not
something an agent can perform by reading source code — the code audit this skill produces can be
**one input** to a risk analysis, not a substitute for one.

## Workforce training and sanctions

§164.308(a)(5) (security awareness training) and §164.308(a)(1)(ii)(C) (a sanction policy for
workforce members who violate policy) are HR/organizational processes with no code-visible
signal. A repo README mentioning "we train employees on security" is not evidence training
occurred or was effective.

## Breach notification

§164.400–414 sets notification timelines and requirements if a breach of unsecured PHI occurs
(generally: affected individuals within 60 days, HHS, and, for breaches affecting 500+
individuals, media notice). If this skill's audit *finds* a live exposure, say so plainly and
flag that the organization should evaluate breach-notification obligations with counsel — this
skill does not make the breach determination itself, and should never advise the user on whether
a specific incident triggers the notification clock.

## Minimum necessary

§164.502(b) requires PHI access, use, and disclosure be limited to the minimum necessary for the
purpose. This has a code-visible component (role-based access design, per `references/technical-safeguards.md`
§164.312(a)) but the *policy judgment* of what's "minimum necessary" for a given role is an
organizational decision this skill doesn't make on the org's behalf — it can flag overly broad
access patterns as a technical finding without asserting what the correct narrower scope is.

## Patient right of access

§164.524 gives individuals a right to access and obtain a copy of their own PHI, generally within
30 days. Whether an in-app data-export feature satisfies this right in full is a legal/product
question outside a code audit's scope — the audit can note whether an export mechanism exists at
all as a factual observation, without asserting it's legally sufficient.

## What to hand the user

When the report reaches this territory, be concrete about the handoff rather than vague:

```
⚪ Needs a human — Business Associate Agreement with [vendor]
   Code shows PHI reaching [vendor] via [file:line]. Confirm a signed BAA covers this
   specific service and workload before this flow continues, or route PHI elsewhere.
   Not something this audit can verify — check with whoever owns vendor contracts.
```

Never fill this gap with a guess dressed as a finding — an unconfirmed BAA status stated as fact
in either direction (compliant or non-compliant) is worse than an honest "needs a human," because
it's the kind of claim someone might act on without re-checking it.
