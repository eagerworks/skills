# GDPR — Lawful Basis and Consent

> Art 6 lawful bases, Art 7 consent conditions, Art 9(2) special-category conditions, and what
> Art 13/14 notices must contain. Read this before accepting a team's claim that "we have
> consent" or "legitimate interest covers this" at face value — both need specific evidence.

## Table of Contents

1. [Art 6 — the six lawful bases](#art-6--the-six-lawful-bases)
2. [Art 7 — conditions for consent](#art-7--conditions-for-consent)
3. [Art 9(2) — special-category conditions](#art-92--special-category-conditions)
4. [Art 8 — children's consent](#art-8--childrens-consent)
5. [Art 13/14 — what a privacy notice must contain](#art-1314--what-a-privacy-notice-must-contain)
6. [Art 21 — right to object, and Art 22 automated decisions](#art-21--right-to-object-and-art-22-automated-decisions)
7. [The ePrivacy / cookie interaction](#the-eprivacy--cookie-interaction)

---

## Art 6 — the six lawful bases

Every processing activity needs at least one of six Art 6(1) bases:

- **(a) Consent** — freely given, specific, informed, unambiguous. See Art 7 below.
- **(b) Contract** — necessary to perform a contract with the data subject, or to take
  pre-contractual steps at their request.
- **(c) Legal obligation** — necessary to comply with a legal obligation the controller is
  subject to.
- **(d) Vital interests** — necessary to protect someone's life, generally limited to emergencies.
- **(e) Public task** — necessary for a task carried out in the public interest or official
  authority (mostly public-sector processing).
- **(f) Legitimate interests** — necessary for the controller's or a third party's legitimate
  interests, "except where such interests are overridden by the interests or fundamental rights
  and freedoms of the data subject."

In code, the basis is a design decision, not a formality — it determines what has to be true
elsewhere. A feature that uses (b) contract as its basis doesn't need a consent checkbox; one
using (a) consent does, and that consent has to meet the Art 7 bar. A common audit finding is a
processing activity with no basis identified anywhere — no consent record, no contract clause, no
documented legitimate-interest assessment — which is itself a 🔴, independent of which basis
would eventually apply.

## Art 7 — conditions for consent

Four conditions, each independently checkable in code or in the flow a user goes through:

- **Art 7(1)** — the controller must be able to *demonstrate* consent was given: a timestamped
  consent record, not just a UI that once showed a checkbox.
- **Art 7(2)** — when consent is bundled into a broader written agreement (e.g. terms of
  service), the consent request must be "clearly distinguishable from the other matters" — a
  single "I agree to the Terms" checkbox covering seven different processing purposes fails this.
- **Art 7(3)** — withdrawal must be "as easy as" giving consent. A consent flow that's one click
  and a withdrawal flow that requires emailing support fails this outright.
- **Art 7(4)** — consent is not "freely given" if the contract or service is made conditional on
  consenting to processing that isn't necessary for that contract — a classic pattern in "accept
  tracking cookies or we won't let you use the free tier" gates.

```
# ❌ wrong — bundled, no free withdrawal, and conditional on service access
if (!acceptedAllCookiesAndMarketing) { blockAccess(); }
# ✅ correct — granular, freely given, symmetric opt-out
consent.essential = true; // no consent needed, strictly necessary
consent.analytics = userChoice.analytics ?? false;
consent.marketing = userChoice.marketing ?? false;
// withdrawal endpoint mirrors the grant endpoint exactly
```

## Art 9(2) — special-category conditions

A processing activity touching Art 9(1) special-category data (see
`references/personal-data-identification.md`) needs an Art 9(2) condition **in addition to** its
Art 6(1) basis — the two aren't interchangeable, and legitimate interest (Art 6(1)(f)) has no
Art 9(2) counterpart, so it can never justify special-category processing on its own (Gotcha 2 in
`SKILL.md`). The most common Art 9(2) condition in commercial software is (a) explicit consent —
note "explicit" is a higher bar than ordinary Art 7 consent, generally read as requiring an
affirmative, specific statement, not just an unambiguous action.

## Art 8 — children's consent

Where consent is the basis for an information society service offered directly to a child, Art 8
requires the child be at least 16 (Member States may lower this to as low as 13) for their own
consent to be valid; below that age, consent must come from whoever holds parental responsibility,
verified using reasonable efforts given available technology. A signup flow with no age gate and
no parental-consent path, offering a consumer service to a general audience that plausibly
includes children, is a 🟡 at minimum pending a human determination of which Member State
threshold applies.

## Art 13/14 — what a privacy notice must contain

Art 13 (data collected from the subject) and Art 14 (data collected elsewhere) require
substantially the same disclosures: controller identity and contact details, the DPO's contact
details where applicable, the purposes and legal basis for each purpose, legitimate interests
pursued if Art 6(1)(f) is used, recipients or categories of recipients, transfer safeguards where
applicable, the retention period or the criteria used to set it, the existence of each data
subject right, the right to withdraw consent, the right to complain to a supervisory authority,
whether providing the data is a legal/contractual requirement, and the existence of any automated
decision-making including profiling.

Auditing this from code means checking the actual privacy-policy page or notice text against this
list — a static file audit can spot a *missing* section (no retention-period statement anywhere,
no mention of a specific third-country transfer that the code clearly performs) more reliably
than it can confirm every clause is legally sufficient; flag gaps, don't attempt a full legal
sign-off on the notice's language.

**A contradiction between the notice and the code is the single highest-value finding this skill
can produce, and it's concretely gradeable — not a ⚪.** The notice is a document, but a specific
factual claim inside it is a checkable fact: "we do not share your data with third parties" next
to a dependency list carrying nine analytics/error-tracking/CRM SDKs is a 🔴, not a "needs a
human." A notice naming three processors when the code calls seven, or promising 12-month
retention with no purge job anywhere in the codebase, are the same pattern. Read the notice early
in the audit (Phase 1/2) specifically so these contradictions surface rather than treating the
notice as out of scope because it's "just a document" — see `SKILL.md` Gotcha 9.

## Art 21 — right to object, and Art 22 automated decisions

Art 21(1) gives the data subject the right to object to processing based on Art 6(1)(e) or (f)
at any time; Art 21(2) gives an unconditional right to object to direct marketing specifically,
and once exercised, Art 21(3) requires that the data no longer be processed for that purpose —
check a marketing/email-preferences flow actually suppresses further sends, not just marks a flag
that a batch job ignores.

Art 22(1) gives a right not to be subject to a decision "based solely on automated processing...
which produces legal effects... or similarly significantly affects" the person, unless it's
necessary for a contract, authorised by law with safeguards, or based on explicit consent — and
even then Art 22(3) requires at minimum the right to human intervention, to express a view, and
to contest the decision. An automated approval/denial flow (credit, eligibility, account
suspension) with no human-review path is a 🔴 if it produces a legal or similarly significant
effect and doesn't fall under one of the three exceptions.

## The ePrivacy / cookie interaction

The requirement to get consent before setting a cookie doesn't come from GDPR — it comes from
**Art 5(3) of the ePrivacy Directive (2002/58/EC, as amended by 2009/136/EC)**, implemented into
each Member State's own national law, so the specific rule in force is the national
implementation, not the Directive text directly. Art 95 of GDPR just avoids piling GDPR
obligations on top of that regime where it already applies — it isn't the source of the consent
requirement, and citing it as such gets the legal basis backwards.

Where GDPR's definition of consent does apply (because the ePrivacy rule requires "consent" as
GDPR defines it), it's the Art 7 standard from above that governs whether it's valid. Two things
broaden this beyond what "cookie banner" suggests: first, Art 5(3) covers "storing of information,
or... gaining access to information already stored" on a user's terminal equipment — not just
cookies. `localStorage`, device fingerprinting, and a mobile SDK's device identifier are all in
scope the same way a cookie is. Second, non-essential storage/access (analytics, advertising,
most third-party embeds) needs affirmative, prior consent regardless of which Art 6 basis would
otherwise apply to the data itself once collected — legitimate interest is not a substitute for
this consent requirement under most Member States' implementations. Grade a page that loads
analytics/ad scripts, or writes to `localStorage` for a non-essential purpose, before any consent
action is taken as a 🔴, independent of whatever the banner's own copy claims.

Note "consent or pay" models (charging for the non-tracked version of a service) are genuinely
unsettled as of this writing, with regulatory guidance still developing — route this pattern to
`references/accountability-and-legal.md` as ⚪ rather than grading it either way.
