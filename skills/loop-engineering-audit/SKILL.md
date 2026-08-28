---
name: loop-engineering-audit
description: >-
  Audits a repository for loop-engineering readiness — whether an AI coding agent can pick up a task, implement it, verify it with local checks, and hand off a PR in an unattended loop — and returns a graded report plus the ordered list of work needed to get there. Use when asked "can agents work on this repo autonomously", "audit this project for Claude Code / Cursor / Codex", "what's missing for AI agents to work here", "prepare this repo for loop engineering / agentic development", "is this codebase agent-ready", or to re-check a project after readiness work was done. The report is printed in chat and saved to docs/loop-engineering-audit.md in the audited project.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Loop Engineering Audit Skill

**Loop engineering** is developing a codebase through autonomous agent loops: an agent takes a well-defined task, implements it, runs the project's own checks until they pass, and opens a PR — with no human in the inner loop. A loop only works when the project gives the agent three things: **context** it can read, **verification** it can run, and **guardrails** that make unattended mistakes cheap. This skill audits a repo for those three things across seven dimensions and produces the **ordered work plan** that closes the gaps.

The audit is **read-only with exactly one write**: the finished report is printed in chat **and** saved to `docs/loop-engineering-audit.md` at the audited repo's root (`docs/` is created if missing; the file is overwritten on re-run so the project keeps one current audit). Nothing else is created, edited, committed, or pushed.

## Discovery — Do This First

Classify the project before grading anything. Every check in `references/rubric.md` depends on knowing the stack and where its commands live.

**1. Detect the stack(s)** from manifests at the root and in workspace globs (`package.json#workspaces`, `pnpm-workspace.yaml`, `turbo.json`, `Gemfile` in subdirs):

| Signal | Stack | Where commands live |
|---|---|---|
| `Gemfile`, `config/application.rb` | Rails | `bin/*`, `Rakefile`, `lib/tasks/`, `.rubocop.yml` |
| `package.json` (+ `tsconfig.json`) | Node / TypeScript | `package.json#scripts`, `turbo.json#tasks` |
| `pyproject.toml`, `requirements*.txt` | Python | `pyproject.toml` `[tool.*]`, `Makefile`, `tox.ini`, `noxfile.py` |
| `go.mod` | Go | `Makefile`, `go test ./...` |
| `Makefile`, `justfile`, `Taskfile.yml` | Any | Task runner — usually the intended entry point |

A monorepo can hold several stacks — audit each package that has its own test/lint surface, and grade the root on how it orchestrates them.

**2. Inventory the agent-facing surface** — read in full, don't skim: `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/*`, `.github/copilot-instructions.md`, `README.md`, `CONTRIBUTING.md`, `.eagerworks/*.json`, `.claude/settings*.json`, `.devcontainer/`, `.github/workflows/*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/ISSUE_TEMPLATE/*`.

**3. Enumerate every verification command** the repo exposes (scripts, rake tasks, make targets, CI steps) and record for each: does it exist, is it documented, is it non-interactive, does it exit non-zero on failure, how long does it take. `references/audit-workflow.md` says which of them you may actually run and how to time them safely.

## What This Skill Does NOT Do

It doesn't fix anything, and it can't see GitHub repo settings (branch protection, required checks) unless `gh` can read them, CI history (flakiness rates), or the team's actual workflow (whether issues are written with acceptance criteria in practice). Those are graded ⚪ **Unverifiable from code** with the exact question a human must answer — never silently skipped, never guessed 🟢.

## The Seven Dimensions

| # | Dimension | An agent loop needs… |
|---|---|---|
| 1 | Agent-facing context | `AGENTS.md`/`CLAUDE.md` that is present, accurate, and states stack, commands, conventions, and forbidden actions |
| 2 | Reproducible environment | One-command, documented setup; pinned toolchain; committed lockfiles; `.env.example`; no hidden manual steps |
| 3 | Fast deterministic verification | Lint, typecheck, test, build each runnable by a single non-interactive command with a correct exit code, in bounded time |
| 4 | Test coverage as a safety net | Real tests where an agent's change would land, so a passing suite actually means something |
| 5 | Task definition surface | Issue/PR templates, acceptance-criteria convention, commit/branch conventions — a well-formed unit of work |
| 6 | CI & merge gates | CI runs the same checks as local; the merge is gated on them; the agent can tell when it's done |
| 7 | Guardrails & safety | Destructive commands fenced, secrets hygiene, hooks, an explicit allow/deny list for agent tooling |

Full checks, decision rules, and Rails / Node-TS / Python examples for each: `references/rubric.md` — read it before grading, it is the authoritative checklist.

## Grades and Verdict

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | The loop cannot run unattended (no non-interactive test command, setup needs a human, secrets required to run tests) |
| 🟡 **Gap** | The loop runs but is slow, unreliable, or leaks human effort (10-minute suite, no lint, undocumented conventions) |
| 🟢 **Ready** | Checked against a concrete rule and clean |
| ⚪ **Unverifiable from code** | Needs a repo setting, a dashboard, or a human to confirm |

Verdict: **Not ready** (any 🔴) → **Partially ready** (🟡 only) → **Ready**. The report's centrepiece is the **Work Plan**: every 🔴 and 🟡 turned into an ordered task with effort (S/M/L), evidence (`file:line` or command), and the concrete change. Format: `references/output-format.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The seven dimensions in full, grade ladder, conservatism rule | `references/rubric.md` |
| Running the audit end-to-end; allowed vs. forbidden commands; timing a test run safely; saving the report | `references/audit-workflow.md` |
| The exact report markdown and the Work Plan table | `references/output-format.md` |
| What a loop-ready `AGENTS.md`/`CLAUDE.md` contains — the checklist dimension 1 grades against | `references/agent-docs.md` |
| The optional `.eagerworks/loop-engineering-audit.json` config | `references/config.md` |

Copyable assets live in `assets/`:
- `assets/audit-report.md` — the report template; fill it in and save it as `docs/loop-engineering-audit.md`
- `assets/AGENTS.example.md` — starter `AGENTS.md` to point at when dimension 1 is 🔴
- `assets/loop-engineering-audit.example.json` — starter config

## Critical Gotchas

1. **One write, nothing else.** The only file you create is the report at `docs/loop-engineering-audit.md` (or `reportPath` from config). Never edit, create, or delete anything else; never `git commit`, `git push`, or open a PR. The Work Plan *describes* fixes — it doesn't apply them.

2. **Never run a command you can't prove is safe.** `bin/setup`, `db:reset`, `prisma migrate`, `docker compose up`, anything with `deploy`, `publish`, or `--force`: read them, don't run them — grade from source and mark timing ⚪. The allowed list is in `references/audit-workflow.md`.

3. **A documented command that doesn't work is a 🔴, not a 🟢.** `README` says `npm test`; `package.json` has no `test` script (or it's `"echo no tests"`) — that's the exact trap that strands an agent. Verify every command exists before crediting it.

4. **Watch mode, prompts, and pagers hang loops.** `jest` without `--ci`/`CI=1`, `vitest` without `run`, `rails console`, `git log` without `--no-pager`, an interactive `bin/setup` — each is a 🔴 unless a non-interactive form is documented.

5. **A grade needs evidence.** Cite the file and line, or the command and its output. "Probably no tests for services" is not a finding; `ls spec/services` returning nothing is.

6. **Don't grade the team's habits 🟢 from templates alone.** A PR template proves a convention exists, not that it's followed — if `gh` can read recent PRs/issues, sample them; otherwise ⚪ with the question to ask.

7. **Zero blockers is a valid result.** Never pad the Work Plan with style preferences to look thorough; an item must trace to a 🔴 or 🟡 in the grades table.

8. **Print the whole report in chat, then save the identical text.** The user asked for both; a chat summary that says "see the file" is not the deliverable.
