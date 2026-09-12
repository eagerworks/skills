---
name: loop-engineering-audit
description: >-
  Audits a repository for loop-engineering readiness — whether an AI coding agent can pick up a task, implement it, verify it with local checks, and hand off a PR in an unattended loop — and returns a graded report, the ordered list of work needed to get there, and an advisory map of which parts of the delivery workflow could be automated plus the specific agent loops this project could create and maintain, each with its risk level, its guardrails, and what to configure to run it. Use when asked "can agents work on this repo autonomously", "audit this project for Claude Code / Cursor / Codex", "what's missing for AI agents to work here", "what can we automate here", "which parts of our workflow could be automated", "which agent loops should we run on this repo", "recommend loops for this project", "can we run agents in parallel on this repo", "is this repo worktree-ready", "prepare this repo for loop engineering / agentic development", "is this codebase agent-ready", or to re-check a project after readiness work was done. The report is printed in chat and saved to docs/loop-engineering-audit.md in the audited project.
metadata:
  author: eagerworks
  version: "1.1.0"
---

# Loop Engineering Audit Skill

**Loop engineering** is developing a codebase through autonomous agent loops: an agent takes a well-defined task, implements it, runs the project's own checks until they pass, and opens a PR — with no human in the inner loop. A loop only works when the project gives the agent four things: **context** it can read, **verification** it can run, **guardrails** that make unattended mistakes cheap, and **room to run more than one loop at once**. This skill audits a repo for those things across eight dimensions and produces the **ordered work plan** that closes the gaps — plus, as advisory output, which parts of the workflow could be automated and which loops the project could actually run.

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

**4. Inventory what is already automated** — every `.github/workflows/*` trigger, `dependabot.yml`/`renovate.json`, `CODEOWNERS`, hooks (`.husky/`, `lefthook.yml`, `.pre-commit-config.yaml`), release tooling (`CHANGELOG.md`, `.changeset/`, `release-please`), deploy config, `.claude/settings.json` permissions, and any existing loop docs (`docs/loops/*` or equivalent). This is the input to the automation map; `references/loop-catalog.md` has the full stage-by-stage list.

## What This Skill Does NOT Do

It doesn't fix anything, and it can't see GitHub repo settings (branch protection, required checks) unless `gh` can read them, CI history (flakiness rates), or the team's actual workflow (whether issues are written with acceptance criteria in practice). Those are graded ⚪ **Unverifiable from code** with the exact question a human must answer — never silently skipped, never guessed 🟢. It also doesn't build the loops it recommends: no workflow file, no permissions file, no playbook — those are described and pointed at, never written.

## The Eight Dimensions

| # | Dimension | An agent loop needs… |
|---|---|---|
| 1 | Agent-facing context | `AGENTS.md`/`CLAUDE.md` that is present, accurate, and states stack, commands, conventions, and forbidden actions |
| 2 | Reproducible environment | One-command, documented setup; pinned toolchain; committed lockfiles; `.env.example`; no hidden manual steps |
| 3 | Fast deterministic verification | Lint, typecheck, test, build each runnable by a single non-interactive command with a correct exit code, in bounded time |
| 4 | Test coverage as a safety net | Real tests where an agent's change would land, so a passing suite actually means something |
| 5 | Task definition surface | Issue/PR templates, acceptance-criteria convention, commit/branch conventions — a well-formed unit of work |
| 6 | CI & merge gates | CI runs the same checks as local; the merge is gated on them; the agent can tell when it's done |
| 7 | Guardrails & safety | Destructive commands fenced, secrets hygiene, hooks, an explicit allow/deny list for agent tooling |
| 8 | Parallel-session readiness | Worktrees that work: gitignored files reproducible, per-worktree DB/ports/containers, hooks that survive a linked worktree, a documented parallel workflow |

Full checks, decision rules, and Rails / Node-TS / Python examples for each: `references/rubric.md` — read it before grading, it is the authoritative checklist.

## Grades and Verdict

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | The loop cannot run unattended (no non-interactive test command, setup needs a human, secrets required to run tests) |
| 🟡 **Gap** | The loop runs but is slow, unreliable, or leaks human effort (10-minute suite, no lint, undocumented conventions) |
| 🟢 **Ready** | Checked against a concrete rule and clean |
| ⚪ **Unverifiable from code** | Needs a repo setting, a dashboard, or a human to confirm |

Verdict: **Not ready** (any 🔴) → **Partially ready** (🟡 only) → **Ready**. The report's centrepiece is the **Work Plan**: every 🔴 and 🟡 turned into an ordered task with effort (S/M/L), evidence (`file:line` or command), and the concrete change. Format: `references/output-format.md`.

**No check in dimension 8 is ever 🔴** — a repo that can't run parallel sessions can still run one loop, so a parallel-session gap can move the verdict from Ready to Partially ready, but never to Not ready.

## What to Automate and Which Loops to Run

Past readiness, the audit answers "what could run itself here?" — as **advisory** output that adds no Work Plan rows and never moves the verdict.

**Automation map.** Classify ten delivery stages — task intake, implementation, local verification, code review, merge, release/changelog, deploy, post-deploy verification, dependency & security maintenance, repo hygiene — as **Automated** / **Assisted** / **Manual** / **Human by design** / **Absent**, each with evidence. Plain words, not grades: a column of all-Automated is not the goal. Merge authorization on production, deploy sign-off, and exploratory QA are **Human by design** and stay that way.

**Risk ladder** — blast radius and reversibility, never "how likely the agent is to be wrong":

| Level | Reach | Made acceptable by |
|---|---|---|
| **L1 Contained** | a branch, a PR, a comment, an issue it opened | human merge is mandatory |
| **L2 Shared-state** | the default branch, the tracker, a tag — landing with nobody accepting it | unbypassable status checks + path allowlist + batch cap + named owner + kill switch |
| **L3 External-effect** | production, published artifacts, real data, third parties | an approval gate the loop can't pass, no production credential, a tested rollback |

Recommend **at most 5**, each with a trigger, a machine-checkable done signal, prerequisites as rubric check ids, guardrails, what to configure, and an owner and kill switch. Never recommend a loop the repo shows no evidence of needing or no ability to run; zero is a valid answer. Catalog, risk ladder in full, selection and ordering rules: `references/loop-catalog.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The eight dimensions in full, grade ladder, conservatism rule | `references/rubric.md` |
| Running the audit end-to-end; allowed vs. forbidden commands; timing a test run safely; saving the report | `references/audit-workflow.md` |
| The exact report markdown and the Work Plan table | `references/output-format.md` |
| What a loop-ready `AGENTS.md`/`CLAUDE.md` contains — the checklist dimension 1 grades against | `references/agent-docs.md` |
| The automation map, the loop risk ladder, the catalog of 10 loops, and the selection rules | `references/loop-catalog.md` |
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

9. **Loop recommendations are advisory.** They never add a Work Plan row, never change a grade, and never move the verdict — the Work Plan still maps 1:1 to 🔴/🟡 checks. A loop that isn't viable yet cites the Work Plan item numbers that unblock it in its `Blocked by` column; that's the only link between the two, and it points one way. Cap: 5.

10. **Never recommend a loop the repo can't run.** No deploy pipeline ⇒ no deploy loop. No committed lockfile ⇒ no dependency loop. No flake evidence ⇒ no quarantine loop. No CI ⇒ no loop has a done signal, so recommend none and name the first one the Work Plan unlocks. A recommendation needs evidence for the same reason a 🟡 does.

11. **Never create a worktree to test worktree readiness.** `git worktree add` writes files and mutates `.git` — that would be a second write. Grade dimension 8 from setup scripts, config, and `.gitignore`; `git worktree list` is read-only and fine. Anything that genuinely needs a live worktree is ⚪ with the question.
