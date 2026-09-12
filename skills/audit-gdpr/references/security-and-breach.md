# GDPR — Security and Breach (Art 32–36)

> Art 32 security of processing, Art 25 data protection by design and by default, Art 5(1)(e)/(f)
> storage limitation and integrity, and the two process obligations a code audit can only
> partially verify: Art 33/34 breach notification and Art 35/36 DPIA.

## Table of Contents

1. [Security of processing — Art 32](#security-of-processing--art-32)
2. [Data protection by design and by default — Art 25](#data-protection-by-design-and-by-default--art-25)
3. [Storage limitation and integrity — Art 5(1)(e) and (f)](#storage-limitation-and-integrity--art-51e-and-f)
4. [Breach notification — Art 33 and Art 34](#breach-notification--art-33-and-art-34)
5. [Data Protection Impact Assessment — Art 35 and Art 36](#data-protection-impact-assessment--art-35-and-art-36)

---

## Security of processing — Art 32

Art 32(1) requires "appropriate technical and organisational measures" scaled to risk, naming
four examples explicitly: (a) pseudonymisation and encryption; (b) ongoing confidentiality,
integrity, availability, and resilience of processing systems; (c) the ability to restore
availability and access to personal data in a timely manner after an incident; (d) a process for
regularly testing and evaluating the effectiveness of these measures. Art 32(2) requires the risk
assessment to account specifically for accidental or unlawful destruction, loss, alteration,
unauthorised disclosure, or unauthorised access. Art 32(4) requires anyone with access under the
controller's or processor's authority to process data only on instruction, unless required by law.

Concrete, checkable evidence:

- **(a) Encryption** — TLS enforced in transit (no plain HTTP fallback for personal-data-bearing
  endpoints), encryption at rest for the primary datastore and backups, confirmed rather than
  assumed from a cloud provider's marketing page (see `references/transfers-and-processors.md`
  for the "opt-in, not automatic" trap this shares with encryption-at-rest defaults).
- **(b) Confidentiality/integrity/availability** — access control on who can query personal data
  directly (not just through the application layer), least-privilege database roles, no shared
  admin credentials.
- **(c) Restoration** — backups exist, are tested (a backup nobody has restored from is not a
  verified control), and restoration time is documented somewhere reachable.
- **(d) Regular testing** — a security testing cadence (pen test, dependency scanning, access
  review) that's more than a one-time setup — grade "we did this once at launch" 🟡, not 🟢.
- **Art 32(4)** — check that database/service credentials given to employees or contractors are
  scoped to instructed purposes, not blanket production access with no audit trail.

**Multi-tenant isolation is a security-of-processing check with a sharp edge.** In a multi-tenant
product, a query missing a tenant scope — `findUnique({ where: { id } })` with no `tenantId`
clause, an unscoped Rails finder, no row-level security policy — lets one customer read another's
personal data. Under Art 4(12), that's "unauthorised... access to[] personal data": a personal
data breach, not merely a bug, the moment it's exploitable, independent of whether it's ever
actually been exploited. Grade a finding like this against Art 32(1)(b) and flag that it may also
trigger the Art 33 assessment described below if there's any indication it was exercised.

## Data protection by design and by default — Art 25

Art 25(1) requires building data-protection measures into the processing itself, "both at the
time of the determination of the means... and at the time of the processing itself" — this is a
design-time and run-time obligation, not a one-time checklist item, and it names pseudonymisation
and data minimisation as example techniques. Art 25(2) is the "by default" half: **by default**,
only personal data necessary for each specific purpose should be processed — covering the amount
collected, the extent of processing, the storage period, and accessibility — and explicitly calls
out that personal data must not "by default" be made accessible to an indefinite number of people
without the individual's intervention (a classic finding: a new user's profile or a shared
document defaulting to public/discoverable rather than private).

In code: a signup form collecting fields nothing in the product currently uses ("just in case"),
a database column collecting more precision than the feature needs (full birthdate stored when
only an age bracket is used), or a new resource's default visibility setting being public rather
than private, are all Art 25(2) findings independent of whether the data is otherwise well
secured. The concrete defaults worth grepping for specifically: a boolean column defaulting to
`true` for something that should be opt-in (`marketing_opt_in BOOLEAN DEFAULT true` in a
migration), a frontend checkbox with `defaultChecked`/`checked` set for a non-essential purpose,
and a new resource (document, profile, share link) that's publicly readable until a user
explicitly locks it down rather than the reverse.

This connects to the other Art 5(1) principles this file doesn't otherwise cover — purpose
limitation (5(1)(b)) and data minimisation (5(1)(c)) — which show up as code the same way: an API
serializer returning every column on a model regardless of what the endpoint needs, a `SELECT *`
into a response payload, an OAuth scope requested broader than the feature uses, or (the most
consequential version) **a full production personal-data export copied into a staging, dev, or
seed environment** with weaker access controls than production has. That last one is a purpose
violation and a minimisation violation and an Art 32 gap simultaneously, and it's common enough in
practice to check for explicitly — look at `db/seeds.rb`, fixture-loading scripts, and any
"refresh staging from prod" job or runbook.

## Storage limitation and integrity — Art 5(1)(e) and (f)

Art 5(1)(e) requires personal data be kept "in a form which permits identification of data
subjects for no longer than is necessary for the purposes for which the personal data are
processed" — an indefinite retention with no deletion job, no TTL, and no documented retention
schedule is a 🟡 at minimum, a 🔴 if the data includes special categories with no retention limit
at all. Art 5(1)(f) (integrity and confidentiality) is the principle Art 32 exists to implement —
cite Art 5(1)(f) for the principle, Art 32 for the concrete measures.

Look for the inverse of the erasure gap in `references/data-subject-rights.md`: not "does erasure
work when requested" but "does data get deleted on a schedule even when nobody asks" — a
retention job, a TTL index, or an explicit documented exception for why a table has none.

## Breach notification — Art 33 and Art 34

Art 33(1): the controller must notify the competent supervisory authority "without undue delay
and, where feasible, not later than 72 hours after having become aware" of a breach, unless it's
"unlikely to result in a risk to the rights and freedoms of natural persons." Art 33(2) requires
a processor to inform the controller "without undue delay" after becoming aware of a breach at
its end. Art 33(3) specifies the notification's required contents (nature of the breach,
categories and approximate number of subjects/records affected, DPO or other contact point,
likely consequences, and measures taken or proposed). Art 33(5) requires the controller to
document every breach — facts, effects, remedial action — regardless of whether it met the
notification threshold, so the supervisory authority can verify compliance on request.

Art 34(1) requires communicating a breach to the affected data subjects directly, "without undue
delay," when it's likely to result in a *high* risk to their rights and freedoms — a higher bar
than the Art 33 authority-notification trigger. Art 34(3) provides exceptions where appropriate
protective measures (e.g. the data was encrypted) render the data unintelligible, where
subsequent measures ensure the high risk is no longer likely, or where individual communication
would involve disproportionate effort (public communication may substitute).

A code audit cannot verify whether an actual breach was handled correctly — this is inherently a
process and incident-response question. What it *can* check is whether the pieces needed to
respond within 72 hours exist at all: is there any documented incident-response procedure, is
there a way to identify which data subjects and how many records were affected by a given
incident (an audit trail sufficient to answer "who was affected"), and is there a named contact
point. Absence of these is a concrete, code-adjacent 🟡/🔴; the breach procedure itself is ⚪ —
see `references/accountability-and-legal.md`.

## Data Protection Impact Assessment — Art 35 and Art 36

Art 35(1) requires a DPIA before starting any processing "likely to result in a high risk to the
rights and freedoms of natural persons," particularly using new technologies. Art 35(3) names
three cases that specifically require one: (a) systematic and extensive evaluation of personal
aspects based on automated processing, including profiling, that produces legal or similarly
significant effects; (b) large-scale processing of special categories of data or criminal
conviction/offence data; (c) systematic monitoring of a publicly accessible area on a large
scale. Art 35(7) specifies the minimum contents: a systematic description of the processing and
its purposes, an assessment of necessity and proportionality, an assessment of risk to data
subjects, and the measures proposed to address that risk. Art 36 requires prior consultation with
the supervisory authority if the DPIA indicates a high risk that isn't sufficiently mitigated.

A code audit can flag that a feature *matches* one of the Art 35(3) triggers (an automated
credit-decisioning flow, a large-scale health-data feature, a public-space monitoring product) —
that's a concrete, evidence-based finding. Whether a DPIA was actually carried out, and whether
it was done well, is not something source code can confirm — route it to
`references/accountability-and-legal.md` as ⚪, citing the specific trigger found.
