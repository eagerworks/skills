# loop-engineering-audit

A portable agent skill that audits a repository for **loop-engineering readiness** — whether an AI coding agent can take a task, implement it, verify it with the project's own checks, and hand off a PR in an unattended loop — and returns the ordered list of work needed to get there. Stack-agnostic, with Rails, Node/TypeScript, and Python examples. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Discovery of the stack(s), workspaces, and every verification command the repo exposes
- Eight readiness dimensions: agent-facing context, reproducible environment, fast deterministic verification, test coverage as a safety net, task definition surface, CI & merge gates, guardrails & safety, and parallel-session readiness (can several agent sessions work the repo at once, in separate worktrees, without colliding)
- A four-level grade per check — 🔴 Blocker / 🟡 Gap / 🟢 Ready / ⚪ Unverifiable from code — rolled up to a mechanical verdict: Not ready / Partially ready / Ready. No parallel-session check is ever 🔴 — it caps throughput, never the verdict
- A **Work Plan**: every blocker and gap as an ordered task with effort (S/M/L), evidence, and the concrete change
- Safe execution rules: lint/typecheck/test/build may be run once, non-interactively, to measure exit codes and runtime; setup, migrations, deploys, installs, and `git worktree add` are never run
- The report is printed in chat **and** saved to `docs/loop-engineering-audit.md` in the audited project (the directory is created if needed) — the one file the skill writes; it never commits or pushes
- A checklist for what a loop-ready `AGENTS.md`/`CLAUDE.md` contains, plus a copyable starter, now including a worktree/parallel-sessions section
- An **automation map** of the delivery workflow — task intake, implementation, local verification, code review, merge, release, deploy, post-deploy verification, dependency & security maintenance, repo hygiene — each classified Automated / Assisted / Manual / Human by design / Absent, with evidence, and an honest stance that some stages (production merge authorization, deploy sign-off, exploratory QA) should stay human
- Up to five **recommended agent loops** from a catalog of ten (issue → PR, backlog refinement, test backfill, mechanical refactor, dependency & advisory patch, CI-failure triage, flaky-test quarantine, agent PR review, docs drift-sync, release notes) — each with a trigger, a done signal, prerequisites as rubric check ids, a blast-radius risk level (L1 contained → L3 external-effect), guardrails, and the concrete config to stand it up
- The loop recommendations are **advisory**: they add no Work Plan rows, change no grade, and never move the verdict — a loop that isn't viable yet cites the Work Plan items that unblock it. Zero recommendations is a valid, expected answer

## Layout

```
SKILL.md                                # hub: discovery, the eight dimensions, grades, automation summary, gotchas (agent entrypoint)
references/
  rubric.md                             # full checklist per dimension, grade ladder, conservatism rule
  audit-workflow.md                     # phase-by-phase procedure, allowed vs. forbidden commands, saving the report
  output-format.md                      # the report markdown: scorecard, Work Plan, automation map, recommended loops, findings, footer
  agent-docs.md                         # what a loop-ready AGENTS.md / CLAUDE.md contains
  loop-catalog.md                       # automation map, loop risk ladder, the 10-loop catalog, selection rules
  config.md                             # .eagerworks/loop-engineering-audit.json schema
assets/
  audit-report.md                       # report template → docs/loop-engineering-audit.md
  AGENTS.example.md                     # starter AGENTS.md to recommend when none exists
  loop-engineering-audit.example.json   # starter config
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration required. To move the report, pin the verification commands, change the runtime budgets, forbid command execution, or disable a dimension that doesn't apply, add `.eagerworks/loop-engineering-audit.json` — see [`references/config.md`](references/config.md) and [`assets/loop-engineering-audit.example.json`](assets/loop-engineering-audit.example.json). Disabled dimensions are always disclosed in the report. The same file turns the loop recommendations off or tightens them — `loops.enabled`, `loops.maxRecommended`, and `loops.maxRiskLevel` (default `2`, so loops that can reach production are never proposed unless you ask). Like a disabled dimension, every one of those is disclosed in the report footer.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill loop-engineering-audit
```
