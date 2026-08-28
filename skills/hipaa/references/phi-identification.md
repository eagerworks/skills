# HIPAA — PHI Identification

> What counts as PHI, the 18 Safe Harbor identifiers, and the two paths to legal de-identification.
> Read this before grading anything "de-identified" or "out of scope" — both calls are wrong more
> often than they're right.

## Table of Contents

1. [PHI vs. PII vs. de-identified](#phi-vs-pii-vs-de-identified)
2. [The 18 Safe Harbor identifiers](#the-18-safe-harbor-identifiers)
3. [Safe Harbor vs. Expert Determination](#safe-harbor-vs-expert-determination)
4. [Limited data sets](#limited-data-sets)
5. [Does HIPAA apply at all?](#does-hipaa-apply-at-all)
6. [Common false negatives](#common-false-negatives)

---

## PHI vs. PII vs. de-identified

**PHI (Protected Health Information)** is individually identifiable health information created,
received, or transmitted by a covered entity or business associate — health status, treatment,
payment for care, tied to an identifiable person. **ePHI** is the same thing in electronic form,
which is what the Security Rule (§164.312, this skill's focus) governs.

PHI is a narrower, stricter concept than general **PII**. A dataset can be sensitive PII (a name
and email) without being PHI at all — PHI requires the health/treatment/payment context. Once
that context exists, HIPAA's de-identification bar is stricter than most teams' intuitive sense
of "anonymized."

```
# ❌ wrong — PII, not automatically PHI: no health context
{ name: "Jane Doe", email: "jane@example.com" }

# PHI — health context makes it PHI even with a first name only
{ name: "Jane", diagnosis: "Type 2 diabetes", provider: "Dr. Smith" }
```

## The 18 Safe Harbor identifiers

Under the Safe Harbor method (§164.514(b)(2)), data stops being PHI only when **all 18** of these
are removed for every individual and (for the household exception) their relatives, employers, and
household members:

1. Names
2. Geographic subdivisions smaller than a state (street address, city, county, precinct); ZIP
   codes narrower than the first 3 digits, *and* any 3-digit ZIP prefix whose combined population
   is ≤ 20,000 must be truncated to `000`
3. All elements of dates (except year) directly related to an individual — birth, admission,
   discharge, death; ages over 89 must be aggregated into a single "90+" category
4. Telephone numbers
5. Fax numbers
6. Email addresses
7. Social Security numbers
8. Medical record numbers
9. Health plan beneficiary numbers
10. Account numbers
11. Certificate/license numbers
12. Vehicle identifiers and serial numbers, including license plates
13. Device identifiers and serial numbers
14. URLs
15. IP addresses
16. Biometric identifiers (fingerprints, voiceprints)
17. Full-face photographs and comparable images
18. Any other unique identifying number, characteristic, or code

**Missing any single one of the 18 means the dataset is still PHI.** There is no partial credit —
"we removed 15 of 18" is not de-identified data, it's PHI with weaker-than-usual protection.

```
# ❌ wrong — "de-identified" but 3 identifiers remain (name-adjacent DOB, ZIP, MRN)
{ patient_id: "MRN-88213", dob: "1979-11-02", zip: "10011", diagnosis: "..." }

# ✅ correct — Safe Harbor: no identifier from the list of 18 remains
{ patient_id: "a91f...", birth_year: "1979", zip3: "100", diagnosis: "..." }
```

## Safe Harbor vs. Expert Determination

Two ways to legally de-identify PHI under §164.514(b):

- **Safe Harbor** — remove all 18 identifiers above. Mechanical, checkable from code.
- **Expert Determination** — a qualified statistical/scientific expert formally determines the
  re-identification risk is very small and documents the analysis and methods used. This is a
  human, out-of-band determination — code review cannot confirm it happened. If a team claims
  Expert Determination covers their data, that claim itself is ⚪ **Needs a human**: ask for the
  determination document, don't take the assertion at face value.

## Limited data sets

A **limited data set** (§164.514(e)) keeps some identifiers — dates and geographic subdivisions
down to town/city/state/ZIP — but strips direct identifiers (names, SSNs, MRNs, etc.) and is
usable for research/public health/operations *only* under a **data use agreement**. It is not
de-identified data and is still subject to Security Rule safeguards; treat it as PHI for audit
purposes unless a data use agreement is confirmed in place.

## Does HIPAA apply at all?

Before grading anything, resolve the boundary case (see `SKILL.md` → Scoping). HIPAA attaches
through a **covered entity or business associate relationship**, not merely because health-shaped
data exists:

| Scenario | HIPAA likely applies | Why |
|---|---|---|
| EHR vendor selling to hospitals/clinics | Yes | Business associate to covered entities |
| Consumer fitness app tracking steps/sleep, no clinical tie-in | Usually no | No covered entity relationship; regulated by FTC/state law instead, not HIPAA |
| Same fitness app adds an integration letting a clinic import patient data | Yes, for that data flow | The clinic's PHI flowing through the app makes the app a business associate for that path |
| Employer wellness program run directly by the employer, no health plan involved | Often no | Employers acting as employers aren't covered entities — but check for group health plan involvement, which does trigger it |
| Telehealth platform | Yes | Covered entity or business associate depending on whether it employs the providers |

When the answer is genuinely ambiguous, say so explicitly in the report rather than picking a
side — this is exactly the kind of legal-boundary call that belongs in
`references/administrative-and-legal.md`'s ⚪ territory, not a confident 🔴/🟢.

## Common false negatives

- **A first name plus a diagnosis** is PHI even with no last name, address, or SSN in sight — see
  the PHI vs. PII example above.
- **Free-text fields** (support tickets, chat transcripts, clinical notes) frequently carry
  identifiers typed inline even when the structured schema has none — audit the *content*, not
  just the column names.
- **Referrer URLs and query strings** can carry a patient ID or email as a parameter, which is
  identifier #14/#15 leaking through a channel nobody thinks of as "the database."
