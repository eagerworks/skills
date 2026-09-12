# hipaa

A portable agent skill that audits a codebase and its infrastructure config against the HIPAA
Security Rule — locating PHI in data models, logs, error trackers, analytics, and outbound
LLM/API calls, then grading findings with `file:line` evidence. Works with Claude Code, Cursor,
GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Scoping the engagement first: covered entity vs. business associate vs. out of scope entirely,
  and locating every place PHI enters, rests, or leaves
- The 18 Safe Harbor identifiers, PHI vs. PII vs. legally de-identified data, and the
  Safe Harbor vs. Expert Determination distinction
- §164.312 technical safeguards — access control, audit controls, integrity, authentication,
  transmission security — mapped to concrete Rails and Node/TypeScript implementations
- The highest-yield audit surface: PHI in application logs, exception trackers, analytics/telemetry
  SDKs, LLM prompts and embeddings, URLs, notifications, and seed/fixture data
- Audit-control depth: read access (not just writes), the 6-year retention rule, tamper resistance,
  and the trap of an audit log that duplicates the PHI it's watching over
- Infrastructure: BAA-eligible service lists for AWS/GCP/Azure, encryption verification commands,
  network isolation, and common third-party vendor BAA gaps
- A severity-graded audit report (🔴 Blocker / 🟡 Risk / ⚪ Needs a human / 🟢 Pass), written to a
  dated file in the audited repo
- A dedicated reference routing administrative and legal obligations (BAAs, training, risk
  analysis, breach notification) to a human, with an explicit not-legal-advice disclaimer

## Scope and limits

This is a **static, read-only audit** — it reads source and config, never inspects live traffic,
and never runs a mutating cloud or infrastructure command. It is not legal advice, not a
substitute for a formal risk analysis or counsel, and there is no such thing as "HIPAA certified"
— HIPAA is a set of legal obligations, not a certification to earn. Administrative and physical
safeguards (BAAs, training, breach notification) are outside what code can verify and are always
routed to a human rather than guessed at — see `references/administrative-and-legal.md`.

BAA-eligible service lists and other cited facts change; the reference files date volatile
information and flag it for re-verification rather than treating it as permanent.

## Layout

```
SKILL.md                            # hub: scoping gate, severity rubric, routing table, gotchas (agent entrypoint)
references/
  phi-identification.md             # the 18 Safe Harbor identifiers, de-identification paths, scope boundary
  technical-safeguards.md           # §164.312 walked through with Rails + Node/TS implementations
  phi-in-code.md                    # logs, error trackers, analytics, LLM calls, seeds — the highest-yield surface
  audit-controls.md                 # §164.312(b) in depth: reads, retention, tamper resistance
  infrastructure.md                 # BAA-eligible services per cloud, encryption/network verification
  audit-workflow.md                 # the 6-phase audit procedure + exact commands + report-writing step
  administrative-and-legal.md       # BAAs, training, risk analysis, breach notification — routed to a human
assets/
  audit-report.md                   # the graded report template — the audit's output shape
  gap-matrix.csv                    # spreadsheet-importable safeguard gap tracker
  phi-inventory.md                  # fillable PHI data-flow inventory
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/)
file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill hipaa
```
