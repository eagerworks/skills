# repo-handoff

A portable agent skill for **inheriting a codebase from another team or company**. It reads the repo from the receiving team's point of view, records everything the code already answers — architecture, setup, deploy, data, third-party services, access, code health, process, operations — and turns everything it can't into a prioritized list of **questions for the previous team**, so the knowledge-transfer meeting is spent on the real gaps. Stack-agnostic, with Rails, Node/TypeScript, and Python probes. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Discovery of the stack(s), every written doc, the git history (authorship, activity, unmerged work), and every external touchpoint the code talks to
- Ten handoff dimensions: overview & architecture, environment & setup, build/test/quality, infrastructure & deploy, data, third-party services & credentials, security & access, code health & known debt, process & history, operations & support
- A four-level grade per check — 🔴 Missing / 🟡 Partial / 🟢 Documented / ⚪ Unverifiable from code — rolled up to a mechanical verdict: Blocked / At risk / Ready
- **Questions for the previous team**: numbered, prioritized P0/P1/P2, each citing the evidence gap, with an `Answer:` line to fill during the handoff — answers are preserved across re-runs
- A service inventory (with secret *names*, never values), a scheduled-jobs table, a bus-factor summary, and an **access & ownership transfer checklist**
- Safe execution rules: lint/typecheck/test and read-only dependency audits may run once; setup, installs, migrations, deploys are never run; secret values are never printed
- The report is printed in chat **and** saved to `docs/repo-handoff.md` in the analyzed project (the directory is created if needed) — the one file the skill writes; it never commits or pushes

## Layout

```
SKILL.md                        # hub: discovery, the ten dimensions, grades, gotchas (agent entrypoint)
references/
  dimensions.md                 # full checklist per dimension, grade ladder, priority ladder, question bank
  handoff-workflow.md           # phase-by-phase procedure, allowed vs. forbidden commands, saving, re-running
  output-format.md              # the report markdown: inventory, findings, questions, risks, access checklist
  config.md                     # .eagerworks/repo-handoff.json schema
assets/
  handoff-report.md             # report template → docs/repo-handoff.md
  repo-handoff.example.json     # starter config
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration required. To move the report, forbid command execution, disable a dimension that doesn't apply, or declare services the receiving team already owns, add `.eagerworks/repo-handoff.json` — see [`references/config.md`](references/config.md) and [`assets/repo-handoff.example.json`](assets/repo-handoff.example.json). Disabled dimensions are always disclosed in the report.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill repo-handoff
```
