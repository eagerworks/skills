# audit-gdpr

A portable agent skill that audits a codebase and its infrastructure config against the GDPR
(Regulation (EU) 2016/679) — resolving territorial scope and the controller/processor role,
locating personal data in data models, logs, error trackers, analytics, and outbound LLM/API
calls, then grading findings with `file:line` evidence. Works with Claude Code, Cursor, GitHub
Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Territorial scope (Art 3) and the controller/processor/joint-controller distinction (Art 4(7)/(8), Art 26)
- Personal data identification: special categories (Art 9), pseudonymisation vs. anonymisation
- Lawful basis (Art 6), consent conditions (Art 7), and special-category conditions (Art 9(2))
- Data subject rights end to end — access, rectification, erasure, restriction, portability, objection (Art 12–22)
- The highest-yield surface: logs, error trackers, analytics, LLM prompts, seeds
- Security of processing (Art 32), data protection by design/default (Art 25), breach notification (Art 33/34), DPIA triggers (Art 35)
- Processor contracts (Art 28), third-country transfers (Art 44–49), and Standard Contractual Clauses
- Accountability items routed to a human — ROPA (Art 30), DPO (Art 37–39), fine tiers (Art 83)

## Scope and limits

This is a **static, read-only audit** of source code and configuration — not a live-traffic
inspection, not a review of an actual Data Processing Agreement's legal text, and not a
substitute for a Data Protection Impact Assessment or counsel. It doesn't determine
controller/processor status with legal certainty in a disputed case, and there is no general
"GDPR compliant" status it can award — findings are graded against specific obligations, not
against an overall pass/fail.

Accountability and legal items the code can't verify (a DPO's actual independence, whether a
balancing test was genuinely performed) are always graded ⚪ **Needs a human**, never assumed
compliant just because a document exists in the repo.

## Layout

```
SKILL.md                              # hub: scoping gate, severity rubric, routing table, gotchas (agent entrypoint)
references/
  personal-data-identification.md     # personal data, special categories, pseudonymisation vs. anonymisation
  lawful-basis-and-consent.md         # Art 6/7/9(2) bases and conditions, Art 13/14 notices, ePrivacy cookies
  data-subject-rights.md              # Art 12-22 mapped to code: soft delete vs. erasure, access vs. portability
  personal-data-in-code.md            # logs, error trackers, analytics, LLM calls, seeds — the highest-yield surface
  security-and-breach.md              # Art 32 security, Art 25 by design/default, Art 33/34 breach, Art 35 DPIA
  transfers-and-processors.md         # Art 28 contracts, Art 44-49 transfers, SCCs, Schrems II
  audit-workflow.md                   # the 6-phase audit procedure + exact commands + report-writing step
  accountability-and-legal.md         # ROPA, DPO, DPIA process, breach procedure, fine tiers — routed to a human
assets/
  audit-report.md                     # the graded report template — the audit's output shape
  gap-matrix.csv                      # spreadsheet-importable obligation gap tracker
  ropa-template.md                    # fillable Art 30 record of processing activities
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/)
file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill audit-gdpr
```
