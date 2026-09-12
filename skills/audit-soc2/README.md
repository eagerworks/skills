# soc2

A portable agent skill for planning and running a SOC 2 readiness effort — intake through gap
analysis to a phased, audit-ready roadmap. Works with Claude Code, Cursor, GitHub Copilot,
Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- A structured intake interview (asked in grouped batches, not a wall of questions) covering
  scope, customers, infrastructure, access, SDLC, data handling, people, and timeline
- Framing the engagement: own company vs. client assessment, Type I vs. Type II, and picking up
  wherever the organization already is
- The Trust Services Criteria (Common Criteria CC1–CC9, plus the four optional categories) and
  when to add an optional category
- System boundary and subservice organization scoping decisions
- A consistent plan output: scope statement, gap matrix, five-phase roadmap, top-5 priorities,
  and stated assumptions
- Required policies — what each must actually contain to survive an auditor, not just exist
- Technical controls mapped to CC5–CC8: access, logging, encryption, vulnerability management,
  backups, change management
- DIY evidence collection and the recurring-control calendar, with an honest note on when a
  compliance platform (Vanta, Drata, Secureframe) starts paying for itself
- Selecting a CPA firm, fieldwork, report anatomy, opinion types, and bridge letters

## Layout

```
SKILL.md                                  # hub: engagement framing, intake, plan format, gotchas (agent entrypoint)
references/
  trust-services-criteria.md              # CC1-CC9 + optional categories, how to decide which apply
  intake-questionnaire.md                 # full question bank with rationale and criterion mapping
  scoping.md                              # system boundary, Type I vs II, observation window, subservice orgs
  policies.md                             # required policy set, what makes a policy audit-ready
  technical-controls.md                   # CC5-CC8 mapped to concrete engineering work
  evidence-and-monitoring.md              # DIY evidence collection, recurring-control calendar
  audit-process.md                        # CPA firm selection, fieldwork, report anatomy, opinion types
  tooling.md                              # DIY stack, and when a compliance platform is worth it
assets/
  intake-questionnaire.md                 # fillable interview for async completion
  gap-matrix.csv                          # spreadsheet-importable control gap tracker
  readiness-roadmap.md                    # phased plan template with a worked example
  policies/                               # 9 policy skeletons, ready to fill in and adopt
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/)
file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Scope and limits

This skill produces readiness guidance — a gap analysis and a roadmap — not legal advice. Only a
licensed CPA firm can perform and issue an actual SOC 2 report; nothing here (or any compliance
platform) substitutes for that. See `references/audit-process.md` for selecting one.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill soc2
```
