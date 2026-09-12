# GDPR — Personal Data Identification

> What counts as personal data, special categories, pseudonymisation vs. anonymisation, and the
> Art 2/3 scope boundary. Read this before grading anything "anonymised" or "out of scope" —
> both calls are wrong more often than they're right.

## Table of Contents

1. [Personal data vs. special categories vs. anonymised](#personal-data-vs-special-categories-vs-anonymised)
2. [Special categories — Art 9](#special-categories-art-9)
3. [Pseudonymisation vs. anonymisation](#pseudonymisation-vs-anonymisation)
4. [Online identifiers count](#online-identifiers-count)
5. [Does GDPR apply at all? — Art 2 and Art 3](#does-gdpr-apply-at-all-art-2-and-art-3)
6. [Common false negatives](#common-false-negatives)

---

## Personal data vs. special categories vs. anonymised

Art 4(1) defines personal data as "any information relating to an identified or identifiable
natural person." That's deliberately broad — it covers anything that lets you single out one
person, directly or by combination with other data you hold. A name is personal data. So is a
device fingerprint, a delivery address with no name attached, or a support ticket with no
identifier field but a unique combination of complaint details that narrows to one customer.

Three buckets, in order of how much protection the data needs:

- **Ordinary personal data** — most of what a typical app stores: name, email, address, order
  history, IP address, session data.
- **Special categories (Art 9)** — a stricter list requiring an Art 9(2) condition on top of an
  Art 6 basis. See the next section.
- **Anonymised data** — genuinely outside GDPR's scope (recital 26): no reasonable means exist,
  for anyone, to re-identify the individual. This is a high bar and most "anonymised" datasets in
  production don't clear it — see [Pseudonymisation vs. anonymisation](#pseudonymisation-vs-anonymisation).

## Special categories — Art 9

Art 9(1) prohibits processing personal data "revealing racial or ethnic origin, political
opinions, religious or philosophical beliefs, or trade union membership," plus genetic data,
biometric data used to uniquely identify a person, health data, and data concerning sex life or
sexual orientation — unless an Art 9(2) condition applies (explicit consent, employment/social
security law, vital interests, a non-profit's own members, data manifestly made public by the
subject, legal claims, substantial public interest, health/social care purposes, public health,
or archiving/research/statistics under Art 89(1)).

Special-category data needs its own Art 9(2) condition **in addition to** an Art 6(1) lawful
basis — the two are not interchangeable, and Art 6(1)(f) legitimate interest cannot substitute
for a missing Art 9(2) condition (see `references/lawful-basis-and-consent.md`).

In code, special-category fields show up more often than teams expect: a "dietary restrictions"
field on an event RSVP form (health), a "pronouns" field (can reveal sexual orientation or gender
identity), a photo upload used for face matching (biometric, if processed "for the purpose of
uniquely identifying a natural person" — a photo stored only for display is not biometric data
under this definition), or a free-text support field where a customer discloses a medical
condition unprompted.

## Pseudonymisation vs. anonymisation

Art 4(5): pseudonymisation is "the processing of personal data in such a manner that the
personal data can no longer be attributed to a specific data subject without the use of
additional information," provided that additional information is "kept separately and is subject
to technical and organisational measures" preventing re-attribution. Recital 26 makes the
consequence explicit: pseudonymised data "should be considered to be information on an
identifiable natural person" — it stays in scope.

A hashed email, a tokenised customer ID, a `user_id` column that joins back to an identity table
— all pseudonymisation, all still personal data, because the org (or anyone with the join key)
can re-identify the person. This is Gotcha 1 in `SKILL.md` for a reason: it's the single most
common false "we anonymised it" claim to correct in an audit.

Anonymisation removes re-identification even in principle — no key, no combination of retained
data, gets back to the individual, for anyone. K-anonymity and aggregation can achieve this if
done correctly; a single retained join key defeats it entirely regardless of how the rest of the
dataset is treated.

```
# ❌ wrong — "anonymised" but user_id still joins back to the identity table
{ user_id: "u_8f2a1c", event: "checkout", email_hash: "a3f9..." }
# ✅ correct — no path back to an identifiable person
{ cohort: "returning_customer", event: "checkout" }
```

## Online identifiers count

Recital 30 and Art 4(1) both cover online identifiers explicitly: IP addresses, cookie
identifiers, device IDs, and advertising IDs are personal data when they can be linked, alone or
in combination, to an identifiable person — which in practice covers most analytics and ad-tech
pipelines. An IP address logged alongside a session ID that later maps to an account is personal
data even if the log line itself has no name or email field.

## Does GDPR apply at all? — Art 2 and Art 3

Material scope (Art 2) excludes processing "in the course of a purely personal or household
activity" and processing by competent authorities for law enforcement purposes (governed by a
separate directive) — most commercial and SaaS processing falls inside scope by default.

Territorial scope (Art 3) is the more common gate in practice — see `SKILL.md` → "Scoping — Do
This First" for the three-basis table (establishment, targeting, monitoring). The audit's first
job is resolving this, not guessing at it from the org's home country.

## Common false negatives

- "We don't store names, just user IDs" — if the user ID joins back to an identity anywhere in
  the system, the data is still personal data (pseudonymisation, not anonymisation).
- "It's B2B data, not consumer data" — GDPR protects natural persons regardless of the business
  context; a work email tied to a named employee is still personal data.
- "It's just an IP address" — see [Online identifiers count](#online-identifiers-count).
- "The field is optional and usually empty" — an optional free-text field still needs to be
  audited for what it collects when filled in, especially for special-category leakage.
- "We deleted the personal fields, kept the rest for analytics" — check what's left: dates,
  ZIP-level location, and a handful of behavioural attributes can still single out an individual
  in combination, especially in a small dataset.
