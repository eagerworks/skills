# GDPR — Accountability and Legal (Needs a Human)

> Not legal advice. This file exists to name what a static code audit cannot verify, so those
> items get routed to a human rather than silently skipped or guessed at. Every item below is
> graded ⚪ **Needs a human** in the audit report — never 🟢, regardless of how thorough a
> document in the repo looks. The one exception: if a document here makes a specific factual
> claim the code contradicts (a ROPA's stated retention period with no purge job anywhere, a
> privacy notice's processor list shorter than the actual dependency list), *that contradiction*
> is a gradeable finding, not a ⚪ — see `SKILL.md` Gotcha 9.

## Table of Contents

1. [Why this is out of scope for a code audit](#why-this-is-out-of-scope-for-a-code-audit)
2. [Accountability — Art 5(2) and Art 24](#accountability--art-52-and-art-24)
3. [Records of processing activities — Art 30](#records-of-processing-activities--art-30)
4. [The Data Protection Officer — Art 37–39](#the-data-protection-officer--art-3739)
5. [The DPIA as a process](#the-dpia-as-a-process)
6. [Breach notification as a process](#breach-notification-as-a-process)
7. [Codes of conduct and certification — Art 40 and Art 42](#codes-of-conduct-and-certification--art-40-and-art-42)
8. [Lead supervisory authority — Art 56](#lead-supervisory-authority--art-56)
9. [Administrative fine tiers — Art 83](#administrative-fine-tiers--art-83)
10. [What to hand the user](#what-to-hand-the-user)

---

## Why this is out of scope for a code audit

Everything in this file is a human process, a document with legal weight, or a determination
that depends on facts no repository fully contains — who actually reviews a DPIA, whether a
balancing test was genuinely weighed or just written to look complete, whether a named DPO has
real independence in practice. Code can confirm a document, a role, or a workflow *exists*; it
cannot confirm the process behind it is real or adequate. Report these as questions to route to
a human — a DPO, counsel, or leadership — never as a passed check.

## Accountability — Art 5(2) and Art 24

Art 5(2) states the controller "shall be responsible for, and be able to demonstrate compliance
with" the Art 5(1) principles — this is the accountability principle the whole regulation rests
on, and it's why this skill's own output (a dated report) is itself a small piece of evidence
toward it, alongside the org's other documentation. Art 24(1) requires implementing measures to
"ensure and to be able to demonstrate" GDPR-compliant processing, "taking into account the
nature, scope, context and purposes of processing" and the risk involved; Art 24(2) requires
data protection policies where proportionate; Art 24(3) allows codes of conduct (Art 40) or
certification (Art 42) as elements of demonstrating this. A code audit can point at evidence
(or its absence) for specific measures — it cannot certify the accountability program as a whole
is adequate.

## Records of processing activities — Art 30

Art 30(1) requires a controller maintain a written record of processing activities including, at
minimum: the controller's (and DPO's) contact details, the purposes of each processing activity,
categories of data subjects and personal data, categories of recipients (including third
countries), any third-country transfers and their safeguards, envisaged erasure time limits where
possible, and a general description of the Art 32 security measures in place. Art 30(2) requires
the processor-side equivalent. Art 30(5) exempts organizations with fewer than 250 employees
*unless* the processing is likely to result in a risk to data subjects, is not occasional, or
involves special-category or criminal-conviction data — a narrow exemption most SaaS products
don't actually qualify for once they process any customer data at meaningful volume.

The audit can confirm whether a ROPA document exists at all (often it doesn't) and use its own
findings from the rest of the audit to populate `assets/ropa-template.md` as a starting draft —
but whether the finished record is complete and accurate is for the org, typically with its DPO
or counsel, to confirm.

## The Data Protection Officer — Art 37–39

Art 37(1) requires designating a DPO when: (a) processing is by a public authority (except courts
acting judicially); (b) the core activities require "regular and systematic monitoring of data
subjects on a large scale"; or (c) core activities involve large-scale processing of special
categories or criminal-conviction data. Art 37(7) requires publishing the DPO's contact details
and communicating them to the supervisory authority. Art 38 requires the DPO be involved "properly
and in a timely manner" in all data-protection matters (38(1)), receive no instructions on how to
carry out their tasks and report to the highest level of management (38(3)), and any other duties
they hold must not create a conflict of interest (38(6)). Art 39(1) lists their tasks: informing
and advising on obligations, monitoring compliance including training and audits, advising on and
monitoring DPIAs, cooperating with the supervisory authority, and acting as its contact point.

Whether an org meets the Art 37(1) trigger is a judgment call the audit can flag evidence for (a
product built around large-scale behavioural tracking, for instance) but not settle definitively.
Whether a named DPO has the independence Art 38 requires, or is doing the Art 39 tasks in
substance rather than title, is entirely a human question.

## The DPIA as a process

`references/security-and-breach.md` covers what triggers a DPIA (Art 35(3)) and what it must
contain (Art 35(7)) — a code audit can flag that a feature matches a trigger. Whether the DPIA
was actually performed, whether it was performed well, and whether Art 36 prior consultation was
needed and happened, are process questions this file routes to a human.

## Breach notification as a process

`references/security-and-breach.md` covers the Art 33/34 legal requirements. A code audit cannot
confirm an incident-response plan would actually hit the 72-hour Art 33(1) window in practice, or
that the org has ever run a breach-notification drill. If no documented procedure exists anywhere
in the repo or linked docs, that absence is itself worth flagging as a gap — but the adequacy of
an existing procedure is for the org to confirm.

## Codes of conduct and certification — Art 40 and Art 42

Art 40 allows associations to draw up codes of conduct approved by a supervisory authority to
help specify GDPR's application to a sector; Art 42 establishes certification mechanisms (data
protection seals and marks) that a controller or processor may seek to demonstrate compliance
with specific processing operations. Both exist and are real, unlike HIPAA's non-existent
certification (`SKILL.md` Gotcha 10) — but they're narrow and scheme-specific. If a repo or its
docs reference a specific certification, note it as a fact to verify (is it current, does it
actually cover the processing in question) rather than treating it as a blanket "GDPR compliant"
claim.

## Lead supervisory authority — Art 56

For a controller or processor with a "main establishment" carrying out cross-border processing,
Art 56(1) designates the supervisory authority of that main establishment as the lead authority
for that processing, coordinating with other concerned authorities under Art 60's cooperation
procedure. Determining which authority is actually the org's lead authority — relevant for where
a breach gets reported, and who an org corresponds with on questions like this audit's findings —
is a legal/organizational fact the audit doesn't determine; it's a good candidate for one of the
questions handed to the user (see below).

## Administrative fine tiers — Art 83

Art 83(4): failures around Articles 8, 11, 25–39, 42, and 43 (broadly: the controller/processor
obligations covered above — DPIAs, records, security, breach notification, DPO, certification)
carry fines up to €10 million or 2% of global annual turnover, whichever is higher. Art 83(5): the
higher tier — up to €20 million or 4% of global annual turnover — covers failures of the core
principles and lawful-basis conditions (Art 5, 6, 7, 9), data subject rights (Art 12–22),
third-country transfer rules (Art 44–49), and non-compliance with a supervisory authority order
(Art 58). This isn't something the audit "checks" — it's context for why a given 🔴 finding
matters, and worth citing next to a Blocker so the severity lands with whoever reads the report.

## What to hand the user

At the end of an audit, this file's items become the questions the report can't answer on its
own. Typical set: Is there a current DPO, and does the Art 37(1) trigger apply to us? Does a ROPA
exist, and if not, can `assets/ropa-template.md` be the starting draft? Which supervisory
authority is our lead authority? Is there a documented breach-notification procedure that's
actually been tested? For any Art 46 safeguard found in the codebase (SCCs with a vendor), is
there an accompanying transfer impact assessment? Present these as a short list at the end of the
report's "Needs a Human" section, each tied to the specific finding that raised it.
