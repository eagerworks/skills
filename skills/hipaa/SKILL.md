---
name: hipaa
description: >-
  Audits a codebase and its infrastructure config for HIPAA Security Rule compliance — locates
  PHI in data models, logs, error trackers, analytics, and outbound LLM/API calls, then reports
  findings graded against §164.312's technical safeguards with file:line evidence. Use when the
  user asks to audit an app for HIPAA, wants to know if they're HIPAA compliant, is building or
  onboarding a healthtech product, mentions PHI, ePHI, covered entity, business associate,
  Business Associate Agreement (BAA), the Security Rule, or the Privacy Rule, asks whether patient
  data is being handled safely, got a question about HIPAA on a security questionnaire, or says
  things like "we're starting to handle patient data" or "is PHI leaking into our logs?" — even
  if they don't name HIPAA directly.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# HIPAA Skill

Audits a codebase and its infrastructure configuration — not a live system, not a legal filing —
against the HIPAA Security Rule (45 CFR §164.312) and reports where PHI is exposed or a required
safeguard is unimplemented. HIPAA is unusual among compliance regimes in that most of its failure
modes are literally visible in source: a logger call next to a `Patient` model, an unscrubbed
error report, a clinical note sent to an LLM with no Business Associate Agreement (BAA) behind it.
This skill finds those. It is not a legal opinion, not a substitute for counsel or a risk
assessment, and there is no such thing as being "HIPAA certified" — HIPAA is a set of legal
obligations enforced by OCR, not a badge to earn.

## Scoping — Do This First

Two questions gate everything below. Get both answered — by inspection, or by asking the user
directly if the code doesn't make it obvious — before producing findings.

**1. Does HIPAA even apply?** Classify the organization's role:

| Role | What it means | Applies here when |
|---|---|---|
| Covered entity | Provides healthcare and transmits health info electronically | The org is a provider, health plan, or clearinghouse |
| Business associate (BA) | Handles PHI on behalf of a covered entity under a BAA | The org's customer is a covered entity and PHI flows to this system |
| Subcontractor of a BA | Handles PHI on behalf of a BA | A sub-processor (hosting, analytics, LLM vendor) the BA has flowed PHI to |
| Neither | No covered entity relationship | Consumer wellness/fitness data with no clinical or insurance tie-in — see `references/phi-identification.md` for the boundary |

Getting this wrong in either direction is the most common mistake: auditing a consumer fitness
app as if it's covered entity data invents findings that don't apply; assuming a B2B healthtech
vendor is "just a tech company" and skipping the audit misses that its customers' BAAs make it a
business associate.

**2. Where does PHI actually flow?** Locate every place PHI enters, rests, or leaves — data
models and migrations, application logs, error trackers, analytics/telemetry SDKs, outbound
LLM/third-party API calls, background jobs, exports, and seed/fixture data. Start with:

```bash
# Identifier-shaped columns/fields near likely PHI models
grep -rniE '(patient|member|client)s?\.(name|email|phone|dob|address|ssn|mrn)' --include='*.rb' --include='*.ts' --include='*.js' .

# Logger/telemetry calls in the same files as PHI-shaped models
grep -rniE '(logger|console\.(log|error)|Sentry\.|analytics\.(track|identify))' app/models lib src 2>/dev/null

# Outbound calls to LLM/AI providers — a common undocumented PHI exit point
grep -rniE '(openai|anthropic|api\.anthropic|generativeai|langchain)' --include='*.rb' --include='*.ts' --include='*.js' .
```

Full identifier catalog and what counts as PHI vs. de-identified: `references/phi-identification.md`.

## What This Skill Does NOT Do

No runtime or network inspection — it reads source and config, not live traffic. It doesn't read
or evaluate an actual BAA's legal text, doesn't determine covered-entity status with legal
certainty, and doesn't substitute for a formal risk analysis, penetration test, or counsel. Items
that need a human to resolve are graded ⚪, never guessed at — see `references/administrative-and-legal.md`.

## Severity Rubric

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | A concrete PHI exposure, or a hard §164.312 requirement with no implementation at all (PHI logged in plaintext, no BAA before PHI reaches a vendor, unencrypted PHI at rest) |
| 🟡 **Risk** | Implemented but weak or unverified — encryption present but not confirmed active, audit logging present but doesn't cover reads, a scrubbing rule that misses a field |
| ⚪ **Needs a human** | Administrative or legal — BAAs, workforce training, risk analysis, breach notification, anything code can't verify |
| 🟢 **Pass** | Checked against a concrete safeguard and clean |

Never grade an administrative safeguard 🟢 — code can confirm a control exists, not that a human
process is actually followed (see Gotcha 6).

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The 18 identifiers, PHI vs. de-identified, Safe Harbor vs. Expert Determination | `references/phi-identification.md` |
| §164.312 technical safeguards mapped to Rails + Node/TS implementations | `references/technical-safeguards.md` |
| Auditing logs, error trackers, analytics, LLM prompts, seeds — the highest-yield surface | `references/phi-in-code.md` |
| §164.312(b) audit controls — what must be logged, retention, tamper resistance | `references/audit-controls.md` |
| BAA-eligible service lists per cloud, encryption verification, network isolation | `references/infrastructure.md` |
| Running the audit end to end, exact commands, writing the report file | `references/audit-workflow.md` |
| BAAs, training, risk analysis, breach notification — routed to a human, not guessed | `references/administrative-and-legal.md` |

Copyable templates live in `assets/`:
- `assets/audit-report.md` — the graded report template; the audit's deliverable shape
- `assets/gap-matrix.csv` — spreadsheet-importable safeguard gap tracker
- `assets/phi-inventory.md` — fillable PHI data-flow inventory (element → storage → vendors → BAA status)

## The Audit in Six Phases

See `references/audit-workflow.md` for the full procedure, exact commands, and how to triage a
large codebase.

1. **Scope** — classify the role, resolve whether HIPAA applies at all (above).
2. **Locate PHI** — data models, logs, error trackers, analytics, LLM calls, exports, seeds.
3. **Technical safeguards** — walk §164.312 against what's found (`references/technical-safeguards.md`).
4. **Audit controls** — confirm access (including reads) is logged and retained (`references/audit-controls.md`).
5. **Infrastructure** — encryption, network isolation, BAA-eligible service usage (`references/infrastructure.md`).
6. **Write the report** — `docs/hipaa-audits/YYYY-MM-DD-<slug>.md` in the audited repo, creating the directory if it doesn't exist. Fill `assets/audit-report.md`: Blockers, then Risks, then Needs-a-human, then a Pass summary, each with `file:line` and a §-cited safeguard.

## Critical Gotchas

1. **"Addressable" is not optional in practice.** §164.312 marks some specifications
   "addressable" rather than "required" — that means the org can implement an equivalent
   alternative and document why, not that it can skip the safeguard. Treat an unimplemented
   addressable spec with no documented alternative as a 🔴, same as a required one.

2. **The 18 Safe Harbor identifiers are broader than names and SSNs.** Dates (birth, admission,
   discharge, death — year alone is usually fine), ZIP codes narrower than 3 digits, device
   identifiers, and full-face photos all count. A dataset with names redacted but birth dates and
   5-digit ZIPs intact is still PHI. Full list: `references/phi-identification.md`.
   ```
   # ❌ wrong — "de-identified" but DOB + ZIP make patients re-identifiable
   { name: null, dob: "1985-03-14", zip: "94107", diagnosis: "..." }
   # ✅ correct — meets Safe Harbor
   { name: null, birth_year: "1985", zip3: "941", diagnosis: "..." }
   ```

3. **A BAA is required before PHI reaches any vendor — including an LLM provider.** Sending a
   clinical note to a model API with no BAA in place is a 🔴 regardless of how good the vendor's
   general security posture is. Flag it as the blocker itself; don't just suggest redacting the
   prompt and call it fixed — the redaction *is* one fix, a BAA covering the vendor is the other,
   and the report should say both are needed until either is confirmed in place.

4. **§164.312(b) audit controls cover reads, not just writes.** A change log that only records
   `UPDATE`/`DELETE` on PHI tables misses the more common real-world incident — an employee
   *viewing* records they had no reason to access. Confirm read access is logged, not just
   mutations. See `references/audit-controls.md`.

5. **"We hashed/removed the name" is not de-identification.** De-identification means meeting the
   full Safe Harbor list or a formal Expert Determination — never a partial redaction the team
   decided was "good enough." Correct this premise plainly when a user asserts it; see
   `references/phi-identification.md`.

6. **Never grade an administrative or legal safeguard 🟢.** Code can confirm a policy document
   exists in the repo; it cannot confirm a human process is actually followed, that a BAA is
   currently signed, or that training happened. Those are always ⚪ — see
   `references/administrative-and-legal.md`.

7. **Never say a client is "HIPAA compliant" or "HIPAA certified."** There is no certification.
   State findings as safeguards implemented or not, with citations — compliance is a legal
   determination this skill doesn't make.

8. **A transitive dependency can leak PHI you never intended to send.** An analytics or
   crash-reporting SDK bundled into a shared package can capture full request payloads —
   including PHI-bearing params — by default. Check SDK-level scrubbing config, not just your own
   logging calls. See `references/phi-in-code.md`.

9. **Facts here go stale.** BAA-eligible service lists, enforcement figures, and specific vendor
   claims change. Before citing one as current, check the date stamp in the relevant reference
   file and flag it for re-verification rather than stating it as permanent fact.
