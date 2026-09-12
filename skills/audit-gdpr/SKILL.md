---
name: audit-gdpr
description: >-
  Audits a codebase and its infrastructure config for GDPR compliance — resolves territorial
  scope and the controller/processor role, locates personal data in data models, logs, error
  trackers, analytics, and outbound LLM/API calls, checks lawful basis and consent, verifies
  data subject rights (access, erasure, portability) are actually implemented end to end, and
  reports findings graded against Regulation (EU) 2016/679 with file:line evidence. Use when the
  user asks to audit an app for GDPR, wants to know if they're GDPR compliant, is expanding into
  the EU/EEA or UK, mentions personal data, data subject, DSAR, right to be forgotten, right to
  erasure, data portability, DPA, DPIA, ROPA, standard contractual clauses (SCCs), cookie
  consent, or a data processing agreement, asks whether they need a DPO, got a question about
  GDPR on a security or vendor questionnaire, or says things like "we're starting to get EU
  customers" or "can we send user data to this LLM provider?" — even if they don't name GDPR
  directly.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# GDPR Skill

Audits a codebase and its infrastructure configuration — not a live system, not a legal filing —
against the General Data Protection Regulation (Regulation (EU) 2016/679) and reports where
personal data is exposed or a required obligation is unimplemented. Like HIPAA, most of GDPR's
failure modes are visible in source: a `deleted_at` column standing in for real erasure, a logger
printing a full user object, special-category data in an analytics payload, an LLM call to a
processor with no Art 28(3) contract behind it. This skill finds those. It is not legal advice,
not a substitute for counsel or a Data Protection Impact Assessment, and there is no general
"GDPR compliant" or "GDPR certified" status to award — GDPR is a set of legal obligations
enforced by supervisory authorities, not a badge.

## Scoping — Do This First

Three questions gate everything below. Get all three answered — by inspection, or by asking the
user directly if the code doesn't make it obvious — before producing findings.

**1. Does GDPR even apply?** Check territorial scope (Art 3):

| Basis | What it means | Applies here when |
|---|---|---|
| Art 3(1) establishment | An establishment of the controller or processor in the Union | The org has an EU/EEA entity, office, or subsidiary, regardless of where processing happens |
| Art 3(2)(a) targeting | Offering goods or services to data subjects in the Union | EU pricing, EU shipping, EU-language marketing, a `.eu`/country-specific site, or EU currency — payment isn't required for this to apply |
| Art 3(2)(b) monitoring | Monitoring the behaviour of data subjects in the Union | Analytics, ad tracking, or profiling that tracks EU-based visitors, regardless of the org's own location |
| Neither | No establishment, no EU targeting, no monitoring of EU behaviour | A domestic-only product with no EU marketing and no EU users tracked |

Getting this wrong in either direction is the most common mistake: assuming "we're a US company"
means GDPR doesn't apply skips it even when the product targets or tracks EU users (Art 3(2));
assuming any EU visitor at all triggers it invents findings for a product with no EU establishment,
no EU targeting, and no monitoring of EU-based behaviour. If none of the three bases apply, say so
and stop — do not produce a graded report.

**2. Controller, processor, or joint controller?** This has no HIPAA analogue and changes which
article set applies:

| Role | Definition | Obligation set |
|---|---|---|
| Controller (Art 4(7)) | Determines the purposes and means of processing | Lawful basis, transparency notices, data subject rights, breach notification to the authority |
| Processor (Art 4(8)) | Processes personal data on behalf of a controller | Art 28 contract terms, sub-processor authorization, assisting the controller — not its own lawful basis |
| Joint controller (Art 26) | Two or more controllers jointly determine purposes and means | A transparent arrangement allocating responsibilities; data subjects can enforce against either |

A B2B product processing its customers' end-users' data is almost always a processor to that
customer — audit it against Art 28, not against the controller's Art 6/13/15 obligations.

**3. Where does personal data actually flow?** Locate every place personal data enters, rests, or
leaves — data models and migrations, application logs, error trackers, analytics/telemetry SDKs,
outbound LLM/third-party API calls, consent/cookie tooling, background jobs, exports, and
seed/fixture data. Start with:

```bash
# Identifier-shaped columns/fields, including special-category (Art 9) candidates
grep -rniE '(user|customer|member)s?\.(name|email|phone|address|dob|ssn|health|religion|ethnicity|orientation)' --include='*.rb' --include='*.ts' --include='*.js' .

# Logger/telemetry calls in the same files as personal-data-shaped models
grep -rniE '(logger|console\.(log|error)|Sentry\.|analytics\.(track|identify))' app/models lib src 2>/dev/null

# Outbound calls to LLM/AI providers — a common undocumented processor relationship
grep -rniE '(openai|anthropic|api\.anthropic|generativeai|langchain)' --include='*.rb' --include='*.ts' --include='*.js' .

# Consent/cookie tooling and region/replica config — transfer and consent surface
grep -rniE '(cookieconsent|onetrust|gdpr|consent_given|eu-west|eu-central|region.*=.*"?(us|eu))' --include='*.rb' --include='*.ts' --include='*.js' --include='*.yml' .
```

Full definitions and the scope boundary: `references/personal-data-identification.md`.

## What This Skill Does NOT Do

No runtime or network inspection — it reads source and config, not live traffic. It doesn't read
or evaluate an actual Data Processing Agreement's legal text, doesn't determine controller/
processor status with legal certainty in a disputed case, and doesn't substitute for a formal
Data Protection Impact Assessment, Legitimate Interest Assessment, or counsel. Items that need a
human to resolve are graded ⚪, never guessed at — see `references/accountability-and-legal.md`.

## Severity Rubric

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | A concrete personal-data exposure, or a hard GDPR requirement with no implementation at all (special-category data logged in plaintext, no processor contract before data reaches a vendor, no lawful basis identified for a processing activity) |
| 🟡 **Risk** | Implemented but weak or unverified — an erasure endpoint that doesn't reach backups or search indexes, a consent banner with a pre-ticked box, encryption present but not confirmed active |
| ⚪ **Needs a human** | Accountability or legal — ROPA, DPIA, DPO designation, a legitimate interest balancing test, breach notification, anything code can't verify |
| 🟢 **Pass** | Checked against a concrete obligation and clean |

Never grade an accountability obligation 🟢 — code can confirm a document exists, not that a
human process is actually followed (see Gotcha 9).

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| Personal data vs. special categories vs. anonymised, pseudonymisation, the Art 2/3 scope boundary | `references/personal-data-identification.md` |
| Art 6 lawful bases, Art 7 consent conditions, Art 9(2) special-category conditions, Art 13/14 notices | `references/lawful-basis-and-consent.md` |
| Art 12–22 data subject rights mapped to code — access, rectification, erasure, portability, objection | `references/data-subject-rights.md` |
| Auditing logs, error trackers, analytics, LLM prompts, seeds — the highest-yield surface | `references/personal-data-in-code.md` |
| Art 32 security of processing, multi-tenant isolation, Art 25 by design/default, Art 5(1) minimisation, Art 33/34 breach, Art 35 DPIA | `references/security-and-breach.md` |
| Art 28 processor contracts, sub-processors (both hiring one and being one), Art 44–49 third-country transfers, SCCs | `references/transfers-and-processors.md` |
| ROPA, DPO, DPIA process, breach procedure, fine tiers — routed to a human, not guessed | `references/accountability-and-legal.md` |
| Running the audit end to end, exact commands, writing the report file | `references/audit-workflow.md` |

Copyable templates live in `assets/`:
- `assets/audit-report.md` — the graded report template; the audit's deliverable shape
- `assets/gap-matrix.csv` — spreadsheet-importable obligation gap tracker
- `assets/ropa-template.md` — fillable Art 30 record of processing activities

## The Audit in Six Phases

See `references/audit-workflow.md` for the full procedure, exact commands, and how to triage a
large codebase.

1. **Scope** — resolve territorial scope and controller/processor role (above).
2. **Locate personal data** — data models, logs, error trackers, analytics, LLM calls, exports, seeds.
3. **Lawful basis & transparency** — walk Art 6/7/9/13/14 against what's found (`references/lawful-basis-and-consent.md`).
4. **Data subject rights** — confirm access, erasure, and portability actually work end to end (`references/data-subject-rights.md`).
5. **Security, transfers & processors** — Art 32 safeguards, Art 28 contracts, Art 44–49 transfers (`references/security-and-breach.md`, `references/transfers-and-processors.md`).
6. **Write the report** — `docs/gdpr-audits/YYYY-MM-DD-<slug>.md` in the audited repo, creating the directory if it doesn't exist. Fill `assets/audit-report.md`: Blockers, then Risks, then Needs-a-human, then a Pass summary, each with `file:line` and an Article-cited obligation.

## Critical Gotchas

1. **Pseudonymised data is still personal data.** Art 4(5) pseudonymisation — a hashed user ID,
   a tokenised email, anything re-identifiable with additional information the org still holds —
   remains fully in scope. True anonymisation, which removes re-identification even in principle,
   is the only thing that leaves scope.
   ```
   # ❌ wrong — "anonymised" but user_id still joins back to the identity table
   { user_id: "u_8f2a1c", event: "checkout", email_hash: "a3f9..." }
   # ✅ correct — no path back to an identifiable person
   { cohort: "returning_customer", event: "checkout" }
   ```

2. **Special-category data needs two locks — legitimate interest only ever holds one.** Every
   processing activity needs an Art 6(1) basis; Art 9 data needs an Art 9(2) condition *in
   addition to* that basis, because Art 9(1) is a separate prohibition only Art 9(2) lifts.
   Art 6(1)(f) legitimate interest can still be the Art 6 basis for special-category data — the
   gap is that there's no legitimate-interest item in the Art 9(2) list, so it can never be the
   *only* thing justifying the processing. Legitimate interest also requires a documented
   balancing test regardless of the data category, is unavailable to public authorities acting in
   their official tasks, cannot be invoked retroactively to cover consent that was never validly
   collected, and a basis can't be swapped once processing has started under another.

3. **Consent's bar is high, and most cookie banners fail it.** Art 7(3) requires withdrawal to be
   as easy as giving consent; Art 7(4) forbids bundling consent into service access; pre-ticked
   boxes and "continuing to browse implies consent" are not valid consent. Storing or accessing
   *anything* on a user's device for a non-essential purpose — not just cookies, also
   `localStorage`, device fingerprints, and mobile SDK identifiers — needs prior consent under
   Art 5(3) of the ePrivacy Directive (as implemented in national law), independently of whichever
   Art 6 basis would otherwise apply to the underlying data.

4. **Soft delete is not erasure — and a HIPAA-trained instinct makes this worse, not better.**
   A `deleted_at` timestamp satisfying a "delete my account" button leaves the row fully intact —
   Art 17 erasure has to reach search indexes, caches, the analytics warehouse, and (Art 19) any
   recipient the data was already disclosed to. Where HIPAA's integrity safeguard (§164.312(c))
   treats soft delete as the *safer* choice, GDPR erasure treats the same pattern as a failure — a
   collision worth naming explicitly rather than assuming HIPAA experience transfers. Backups are
   the one exception with a practical answer: expiring them on their normal rotation schedule with
   the deletion re-applied on any restore is an accepted approach, not a 🔴 — grade backups
   pragmatically, and reserve 🔴 for gaps the org could close today (the index, the cache, the
   warehouse). Also check Art 17(3): a documented legal-retention obligation (tax records, an
   active legal claim) is a valid reason to keep the minimum necessary data — don't demand erasure
   of data the org is legally required to hold.

5. **Access (Art 15) and portability (Art 20) are not the same request.** Access covers all
   personal data plus the Art 15(1)(a)–(h) metadata (purposes, recipients, retention criteria,
   safeguards for transfers); portability covers only data the subject provided, and only when
   processing is based on consent or contract *and* carried out by automated means. A single JSON
   export endpoint that ignores this distinction satisfies neither request correctly.

6. **A processor needs an Art 28(3) contract before data flows — including an LLM provider.**
   Sending personal data to a model API with no Data Processing Agreement in place is a 🔴
   regardless of the vendor's general security posture. Prompt redaction is one fix, the contract
   is the other — flag both as open until confirmed, don't call redaction alone sufficient. And
   if the LLM vendor trains on submitted inputs by default, it's processing beyond the
   controller's instruction and becomes a controller for that processing under Art 28(10) — check
   the account's training/retention setting, not just whether a contract exists.

7. **If the audited org itself is the processor, most of this checklist flips.** Resolved in
   Gate 2, but easy to lose sight of mid-audit: a processor has no Art 6/9 lawful-basis question
   of its own (that's the controller's job), no Art 13/14 notice duty to the controller's data
   subjects, and no Art 33(1) 72-hour clock to a supervisory authority — only an Art 33(2)/28(3)(f)
   duty to notify *the controller* without undue delay. What it does need: an Art 28(3) contract
   with each controller, Art 28(2) authorization before adding a sub-processor, a way to assist
   the controller with data subject rights requests (Art 28(3)(e) — a tenant-scoped export/delete
   capability is the concrete code check), deletion or return of data at contract end
   (Art 28(3)(g)), and its own Art 30(2) processing record. See
   `references/transfers-and-processors.md` → "If the audited org is the processor."

8. **A third-country transfer includes remote access, not just a data copy.** A support engineer
   or contractor accessing production data from outside the EU/EEA is a transfer under Art 44,
   even with no data physically moved. Standard Contractual Clauses alone are not sufficient
   after *Schrems II* — a transfer impact assessment of the destination country's laws goes with
   them. See `references/transfers-and-processors.md`.

9. **Never grade an accountability obligation 🟢 — but a contradiction with one is still
   gradeable.** ROPA (Art 30), DPIA (Art 35), DPO designation (Art 37), the legitimate interest
   balancing test, the breach notification procedure (Art 33) — code can confirm a document or a
   role exists, never that the process behind it is real. Always ⚪ on the process itself. But
   when a document makes a specific factual claim the code can check — a ROPA stating 12-month
   retention against no purge job anywhere, a privacy notice listing three processors against
   seven found in the dependency list — the *contradiction* is a concrete finding, not a ⚪; grade
   it on the evidence. See `references/accountability-and-legal.md`.

10. **Never say a client is "GDPR compliant," and date-stamp adequacy claims.** Unlike HIPAA,
    Art 42 certification schemes do exist, but they're scheme-specific and narrow — passing one is
    not general compliance. Cross-border transfer facts (adequacy decisions, the status of
    standard contractual clauses, a specific vendor's current certifications) change: check the
    date stamp in `references/transfers-and-processors.md` before citing one as current, and flag
    it for re-verification rather than stating it as permanent fact.
