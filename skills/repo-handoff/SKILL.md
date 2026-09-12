---
name: repo-handoff
description: >-
  Analyzes a codebase being inherited from another team or company and produces a handoff report — what the code reveals about architecture, setup, deploy, data, third-party services, access, code health, process, and operations — plus a prioritized list of questions the previous team must answer before they leave. Use when asked "we're taking over this repo", "prepare the handoff for this project", "what do we need to ask the previous team", "audit this inherited codebase", "knowledge-transfer checklist for this repo", or to re-run after the previous team has answered. The report is printed in chat and saved to docs/repo-handoff.md in the analyzed project.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Repo Handoff Skill

A **handoff** is the moment a codebase changes hands: one team stops owning it, another starts. Most of what the outgoing team knows is not in the repo — which account owns the DNS, why that migration was never run, which cron job must not overlap, who gets paged. This skill reads the repo from the **receiving team's** point of view, records everything the code *does* answer, and turns everything it *doesn't* into a concrete, prioritized list of questions for the previous team — so the knowledge-transfer meeting is spent on the real gaps, not on things `grep` could have told you.

The analysis is **read-only with exactly one write**: the finished report is printed in chat **and** saved to `docs/repo-handoff.md` at the analyzed repo's root (`docs/` is created if missing; the file is overwritten on re-run, preserving answers already filled in). Nothing else is created, edited, committed, or pushed.

## Discovery — Do This First

Classify the project before analyzing anything. Every check in `references/dimensions.md` depends on knowing the stack and where things live.

**1. Detect the stack(s)** from manifests at the root and in workspace globs (`package.json#workspaces`, `pnpm-workspace.yaml`, `turbo.json`, `Gemfile` in subdirs, `pyproject.toml`, `go.mod`, `*.csproj`, `pubspec.yaml`, `app.json`/`eas.json`).

**2. Inventory the written knowledge** — read in full, don't skim: `README*`, `CONTRIBUTING*`, `docs/**`, `AGENTS.md`, `CLAUDE.md`, `CHANGELOG*`, ADRs, wikis checked into the repo, `.env.example`, `docker-compose*.yml`, `Dockerfile*`, `.github/**`, `.gitlab-ci.yml`, `config/deploy*`, `terraform/`, `infra/`, `Procfile`, `fly.toml`, `render.yaml`, `vercel.json`, `app.yaml`.

**3. Mine the history** — `git log`, `git shortlog -sn`, `git branch -a`, tags, open PRs/issues via `gh` if available. Authorship concentration, last-activity dates, and unmerged branches are the handoff's early-warning signals.

**4. Enumerate every external touchpoint** — anything the code talks to that lives outside the repo: databases, queues, SaaS APIs, payment providers, email/SMS, auth providers, object storage, CDNs, analytics, error tracking, app stores, domains. Each one is an **account someone owns** and a **credential someone holds** — the two things most often lost in a handoff.

## What This Skill Does NOT Do

It doesn't fix, refactor, or document the code for the previous team — it documents what is *known* and what is *unknown*. It can't see dashboards, cloud consoles, password managers, chat history, or contracts; anything that lives there is graded ⚪ **Unverifiable from code** and becomes a question — never guessed, never silently skipped. It never prints the value of a secret it finds, only its location.

## The Ten Dimensions

| # | Dimension | The receiving team needs to know… |
|---|---|---|
| 1 | Overview & architecture | What the system does, for whom, its major components and boundaries, the key domain models, and the decisions behind them |
| 2 | Environment & setup | How to get a working dev environment from a clean machine, with no hidden manual steps |
| 3 | Build, test & quality | Which commands build/lint/test it, whether they pass today, and where the suite is thin or flaky |
| 4 | Infrastructure & deploy | Where it runs, how it gets there, how to roll back, and which environments exist |
| 5 | Data | Which datastores exist, migration state, seeds, backups, restore procedure, PII and retention |
| 6 | Third-party services & credentials | Every external service, which account owns it, where its secrets live, and what must be transferred or rotated |
| 7 | Security & access | Auth model, secrets hygiene (incl. git history), dependency vulnerabilities, and the full list of accesses to transfer |
| 8 | Code health & known debt | Hot spots, TODO/FIXME/HACK density, dead code, outdated dependencies, and the debt the previous team already knows about |
| 9 | Process & history | Branching model, release cadence, bus factor, unfinished work on branches, open PRs/issues |
| 10 | Operations & support | Monitoring, logging, alerting, runbooks, on-call, SLAs, scheduled jobs, and known recurring incidents |

Full checks, decision rules, and the canonical **question bank** per dimension: `references/dimensions.md` — read it before grading; it is the authoritative checklist.

## Grades and Verdict

| Grade | Meaning |
|---|---|
| 🔴 **Missing** | The knowledge is not in the repo and the system can't be safely operated without it — it lives only in the previous team's heads |
| 🟡 **Partial** | Some of it is written down, but incomplete, outdated, or contradicted by the code |
| 🟢 **Documented** | Checked against a concrete rule; the repo answers it and the code agrees |
| ⚪ **Unverifiable from code** | Lives in a console, dashboard, contract, or person — needs a human to confirm |

Verdict is mechanical: **Blocked** (any 🔴 in dimensions 2, 4, or 6 — you can't run, ship, or keep the lights on) → **At risk** (any other 🔴, or any 🟡) → **Ready**. The report's centrepiece is **Questions for the previous team**: every 🔴, 🟡, and ⚪ turned into a numbered question with a priority (P0 / P1 / P2), the evidence gap that motivates it, and an empty `Answer:` line to be filled during the handoff. Format: `references/output-format.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The ten dimensions in full, grade rules, question bank per dimension | `references/dimensions.md` |
| Running the analysis end-to-end; allowed vs. forbidden commands; saving the report; re-running after answers | `references/handoff-workflow.md` |
| The exact report markdown: scorecard, findings, questions, risks, access checklist | `references/output-format.md` |
| The optional `.eagerworks/repo-handoff.json` config | `references/config.md` |

Copyable assets live in `assets/`:
- `assets/handoff-report.md` — the report template; fill it in and save it as `docs/repo-handoff.md`
- `assets/repo-handoff.example.json` — starter config

## Critical Gotchas

1. **One write, nothing else.** The only file you create is the report at `docs/repo-handoff.md` (or `reportPath` from config). Never edit, create, or delete anything else; never `git commit`, `git push`, or open a PR. Recommendations *describe* fixes — they don't apply them.

2. **Never run a command you can't prove is safe.** `bin/setup`, `db:migrate`, `prisma migrate`, `docker compose up`, `terraform apply`, anything with `deploy`, `publish`, `--force`: read them, don't run them. The allowed list is in `references/handoff-workflow.md`.

3. **Never print a secret's value.** If `.env`, a config file, or git history contains a credential, report `file:line` (or the commit hash) and say "rotate on handoff" — never the value itself, not even partially.

4. **A question must trace to an evidence gap.** "How does deploy work?" is not a question when `.github/workflows/deploy.yml` answers it. Only ask what the repo could not tell you, and say what you looked at. Padding the question list wastes the previous team's goodwill and buries the questions that matter.

5. **Every external service is two questions: who owns the account, and where is the credential.** Stripe, Sentry, SES, S3, Twilio, Firebase, the App Store, the domain registrar, the DNS host — each is a P0 unless the repo shows the receiving org already owns it.

6. **Don't grade a README 🟢 without checking it against the code.** A setup section that references a script that no longer exists, or a deploy doc for a platform the code left two years ago, is 🟡 with the contradiction as evidence — and a question.

7. **Bus factor is a finding, not a judgment.** `git shortlog -sn` showing one author for 90% of commits is reported as fact with a P0 question ("Will <role> be reachable after <date>?"), never as criticism of the previous team.

8. **Re-runs preserve answers.** If `docs/repo-handoff.md` already exists with filled-in `Answer:` lines, carry each answer over to the matching question (matched by question text) before overwriting. Losing the previous team's answers is the one thing worse than not asking.

9. **Print the whole report in chat, then save the identical text.** The user asked for both; a chat summary that says "see the file" is not the deliverable.
