---
name: audit-open-source-repo
description: >-
  Audits a repository for contributor readiness — whether an outside contributor who has never seen the codebase can discover the project, set it up without the team's secrets, find scoped work, build a feature at the maintainers' quality bar, and get it merged — and returns a graded report plus an ordered, prioritized plan of the files and config changes needed to get there. The core question is whether the project's guidelines are machine-enforced (linters, formatters, hooks, CI), not just written down, since an unenforced guideline is one a newcomer will break and a reviewer will re-teach every PR. Use when asked "is this repo ready for contributors", "audit our open-source repo", "why aren't we getting good contributions", "prepare this project for external contributors", or "review our CONTRIBUTING/README/community health". Distinct from loop-engineering-audit, which asks whether an unattended agent, not a human, can develop the repo. The report is saved to a dated file under docs/open-source-audits/.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Audit Open Source Repo Skill

**Contributor readiness** is not "does a `CONTRIBUTING.md` exist" — it's whether a stranger can walk the whole path: discover the project, set it up with none of the team's secrets, find work that's actually scoped, build a *feature* the way the team would have built it, prove it's correct, and get it reviewed and merged. The load-bearing insight is that **a guideline that isn't executable is a guideline the contributor will break and the reviewer will re-teach on every PR** — so the heart of this audit is whether the project's standards are enforced by a command, not just written in prose, and whether the orientation docs are actually true. This skill grades that across ten dimensions and produces the **Contributor Readiness Plan** — the ordered, prioritized list of concrete artifacts that closes the gaps — plus a **first contribution dry run** that traces the exact path a newcomer takes and reports the first point it breaks.

The audit is **read-only with exactly one write**: the finished report is printed in chat **and** saved to `docs/open-source-audits/YYYY-MM-DD-<slug>.md` at the audited repo's root (`docs/open-source-audits/` is created if missing). Each run is its own dated file — never overwritten, `-2` appended on a same-day collision — so the repo accumulates a history a maintainer can diff to see readiness improve. Nothing else is created, edited, committed, or pushed; the Plan *describes* the `CONTRIBUTING.md`, the CI workflow, the issue templates it wants — it never writes them.

## Discovery — Do This First

Every check in `references/dimensions.md` depends on knowing the audience and the stack before you grade anything.

**1. Settle the audience.** Read `project.audience` from config (`references/config.md`); absent that, infer from evidence: `gh repo view --json isPrivate` (if `gh` can read it), whether the README/LICENSE address the public, a company copyright header. Public OSS is the default. Under `audience: "internal"`, the OSI-licence, public-discoverability and public-vulnerability-disclosure checks become **n/a** — excluded and disclosed in the footer, never graded 🔴 — and the contribution flow is graded branch-based rather than fork-based.

**2. Detect the stack** the same way `loop-engineering-audit` does — manifests at the root and in workspace globs (`package.json`, `Gemfile`, `pyproject.toml`, `go.mod`, `Cargo.toml`) — because dimension 7's linter/formatter/typecheck checks and dimension 4's toolchain-pinning checks are stack-specific.

**3. Inventory the contributor-facing surface**, read in full: `README.md`, `LICENSE`, `CONTRIBUTING.md` (root, `.github/`, `docs/` — all three platform lookup paths), `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`, `GOVERNANCE.md`, `CODEOWNERS`, `.github/ISSUE_TEMPLATE/*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/workflows/*`, linter/formatter/typecheck configs, `.editorconfig`, hook configs (`.husky/`, `lefthook.yml`, `.pre-commit-config.yaml`), `.env.example`, `docker-compose*.yml`, pinned-toolchain files.

**4. Mine `git`/`gh` history** for dimension 10 and the "practice vs. template" checks in dimensions 2 and 5 — `git log --oneline -30`, `git shortlog -sn --no-merges` over 12 months, `gh pr list --state merged --json author,createdAt,mergedAt,reviews`, `gh issue list --json title,body,labels,createdAt`, `gh label list`, `gh release list`. A `gh` call that 403s/404s makes the corresponding check ⚪ with the question, never a guess.

## What This Skill Does NOT Do

It doesn't fix anything, write a single missing file, open an issue, label one, or comment. It can't see private repo settings (branch protection, required checks, "require approval for first-time contributor workflow runs") unless `gh` can read them — those are ⚪ **Unverifiable from code** with the exact question, never silently skipped, never guessed 🟢. It never recommends adopting a CLA. It says nothing about architecture quality, security posture, or whether the codebase is *good* — a repo with excellent onboarding and a mediocre codebase is correctly graded 🟢 Contributor-ready, because that's the question this audit answers.

## The Ten Dimensions

| # | Dimension | *Question it answers* |
|---|---|---|
| 1 | Discoverability & first impression | Can a stranger tell what this is, whether it's alive, and whether they may legally change it? |
| 2 | The contribution contract | Does the repo state how an outsider proposes work — and what will actually be accepted? |
| 3 | Community health & governance | Does a contributor know who decides, where to ask, and what happens when something goes wrong? |
| 4 | Time to first build | Can someone with none of the team's secrets get from `git clone` to a green test run? |
| 5 | Task surface | Is there something specific, scoped, and unclaimed they could pick up today? |
| 6 | Code orientation for feature work | Can they find where a feature goes, and build it the way the team would have? |
| 7 | **Guidelines as an executable spec** | Are the standards enforced by a command they can run, instead of by a reviewer's memory? |
| 8 | Self-verification before the push | Can they prove the feature is done and correct before a maintainer looks at it? |
| 9 | Fork-safe CI & merge gates | Does the gate actually run — and pass — on a pull request from a fork? |
| 10 | Review & release loop | If they open a good PR, does anything happen to it, and does merged work reach users? |

Full checks, evidence rules, and the exhaustive 🔴 list: `references/dimensions.md` — read it before grading, it is the authoritative checklist. Dimension order **is** the newcomer's journey; the dry run and the Plan's ordering rule both reuse it.

## Grades and Verdict

| Grade | Meaning |
|---|---|
| 🔴 **Barrier** | A first-time outside contributor is stopped, or sent down a path that cannot succeed |
| 🟡 **Friction** | They get through, but it costs them or the maintainer time the repo could have saved — or lets a feature land below the project's own bar |
| 🟢 **Clear** | A concrete rule was checked and satisfied |
| ⚪ **Unverifiable from code** | Lives in a repo setting, a dashboard, a private channel, or a person — always carries the exact question |

Verdict: **🔴 Closed to contributors** (any 🔴) → **🟡 Open with friction** (🟡 only) → **🟢 Contributor-ready**. ⚪ never moves it. **Every 🔴 is verdict-blocking** — unlike the capped dimensions below, the 🔴 conditions are enumerated exhaustively in `references/dimensions.md`, and nothing outside that list may ever be graded 🔴.

**No check in dimension 5 or 10 is ever 🔴.** Dimension 5 grades *invitation*, not possibility — a contributor who brings their own itch can still contribute to a repo with zero `good first issue`s. Dimension 10 is the only dimension graded from **people's history**, not artifacts, and turning a median-time-to-review into a blocking verdict is the failure mode most likely to make a maintainer discard the whole report. When any 10.x check is 🟡, the report's lead paragraph must open with the stewardship numbers (median time to first review, external PRs merged, oldest unanswered PR) — the cap is not a way to hide a dead project.

The report's centrepiece is the **Contributor Readiness Plan**: every 🔴 and 🟡 turned into an ordered, P0/P1/P2-prioritized row naming the concrete artifact to produce, its effort (S/M/L), and its evidence. A **first contribution dry run** then traces the journey stage by stage against grades already assigned and names the exact stage where it stops. Format for both: `references/output-format.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The ten dimensions in full, the exhaustive 🔴 list, the two capped dimensions, conservatism rule | `references/dimensions.md` |
| Running the audit end-to-end; allowed vs. forbidden commands; writing and comparing dated reports | `references/audit-workflow.md` |
| The exact report markdown and the Contributor Readiness Plan table | `references/output-format.md` |
| The eight-stage first contribution dry run: what it inspects, what it may run, its honesty rules | `references/first-contribution.md` |
| The optional `.eagerworks/audit-open-source-repo.json` config | `references/config.md` |

Copyable assets live in `assets/`:
- `assets/readiness-report.md` — the report template; fill it in and save it as a dated file under `docs/open-source-audits/`
- `assets/audit-open-source-repo.example.json` — starter config
- `assets/CONTRIBUTING.template.md` — starter to point at when dimension 2 is 🔴
- `assets/PULL_REQUEST_TEMPLATE.md` — starter for check 5.4
- `assets/ISSUE_TEMPLATE/bug_report.yml`, `assets/ISSUE_TEMPLATE/feature_request.yml` — starters for checks 5.1–5.2
- `assets/SECURITY.template.md` — starter for check 3.2

## Critical Gotchas

1. **One write, nothing else.** The only file you create is the dated report under `docs/open-source-audits/` (or `reportPath` from config). Never fork, clone, branch, commit, push, open an issue or PR, post a comment, or add a label — including the `good first issue` labels the Plan says are missing. The Plan *describes* the `CONTRIBUTING.md` it wants; writing it is the maintainer's job.

2. **Never run setup to find out whether setup works.** `bin/setup`, `make bootstrap`, `npm install`, `bundle install`, `docker compose up`, `db:migrate`: read them and resolve every token they name against the tree. A setup path is graded by **resolution, not execution** — that's what makes dimension 4 safe to audit. Already-installed dependencies may be used for dimension 7/8 commands; missing ones make those checks ⚪, never 🟡.

3. **A CONTRIBUTING.md that contradicts the code is worse than none.** The contributor follows it, hits a command that doesn't exist, concludes nobody is home, and leaves — and unlike a missing document, nothing warned them. Extract every backticked command, path, script and env var from CONTRIBUTING and the README's setup section and resolve each against the tree; a named command that doesn't exist is 🔴 no matter how thorough the prose reads.

4. **A template is not a practice.** An issue form, a PR checklist, a commit convention, a "we review within 48 hours" sentence each prove only that someone intended it on the day it was committed. Sample reality with `gh`/`git`. An unused template is 🟡 with the ratio as evidence; unreadable is ⚪. Never grade a practice 🟢 from the file that asks for it alone.

5. **Never grade maintainer behaviour from a document.** "We aim to review within 48 hours" is the *threshold*; `gh pr list --json createdAt,reviews` is the *grade*. Dimension 10 is history-only, and its findings are numbers — "median first review 11 days across the last 12 PRs" — never a characterization of the people. When `gh` can't read, it's ⚪ with the question, not a charitable 🟢.

6. **Never invent an entry point.** Don't name a `good first issue` that doesn't exist, describe an issue you didn't open and read, or cite an issue number because it looked plausible. Zero entry-point issues is a finding with a Plan row ("scope and label these three existing issues"), not a gap papered over with a fabricated example. Same for a roadmap, a chat channel, or a maintainer's name.

7. **Don't recommend a CLA.** A CLA is a legal and political choice with real costs — it deters contributors and requires an entity to hold copyright. This audit never recommends adopting one and never treats its absence as a gap. It checks **consistency only**: a CLA/DCO check running on PRs with no document naming it is 🔴; a documented sign-off nothing enforces is 🟡; a repo with neither is 🟢 — "no agreement required" is a complete, common, and usually correct answer.

8. **An internal repo is not a public one — and "public" is not "accepting PRs."** Settle `project.audience` before grading dimensions 1 and 3 (Discovery step 1). Under `internal`, a missing OSI LICENSE, missing topics, and a missing public code of conduct are n/a, excluded and disclosed in the footer, never 🔴, and the flow is graded branch-based. Separately: if the repo explicitly states it doesn't accept external contributions, say so in the header and grade only the dimensions that still apply, rather than producing a plan to open up a project that chose not to be open.

9. **The dry run traces; it does not role-play.** Every result in `references/first-contribution.md` is derived mechanically from grades already assigned, every cell is something read, run, or measured, and the one thing it may say about a human is where the documented path stopped resolving. It adds no finding, no Plan row, and moves no grade — the moment it starts narrating feelings it has stopped being evidence.

10. **Absence is only a finding when the project is the kind that needs the thing.** No `CHANGELOG.md` in a repo that's never released, no migration convention with no database, no screenshot in a headless library, no `GOVERNANCE.md` for a one-person project that says it is one, no CLA anywhere: each is 🟢 or n/a, not 🟡. A 🟡 must cite the thing the project *has* that makes the gap real, never just "no docs for X."

11. **This is not `loop-engineering-audit`.** That skill asks whether an unattended *agent* can develop the repo — agent instruction files, non-interactive command shape, worktree isolation, guardrails against unsupervised mistakes. This skill asks whether an outside *human* can land a quality feature — README/LICENSE/CONTRIBUTING/governance, the fork-based path, machine-enforced guidelines, orientation docs, the task surface, and the review/release loop measured from history. Where they touch the same artifact (CI, tests, conventions) they grade a different property of it — see `references/dimensions.md` → "Boundary with loop-engineering-audit". Never produce both reports from one run, never cross-cite a check id, never let one skill's plan absorb the other's rows.

12. **🟢 here means "a stranger can land a good feature," not "the code is good."** This audit says nothing about architecture quality, security posture beyond `SECURITY.md`'s reporting route, or test quality beyond documented conventions. Don't smuggle code review into the Plan.
