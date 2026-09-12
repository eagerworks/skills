# audit-open-source-repo

A portable agent skill that audits a repository for **contributor readiness** — whether an outside contributor who has never seen the codebase can discover the project, set it up without the team's secrets, find scoped work, build a feature at the quality bar the maintainers expect, and get it reviewed and merged — and returns the ordered, prioritized plan of concrete files and config changes needed to get there. Stack-agnostic, with Rails, Node/TypeScript, and Python examples. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Ten contributor-readiness dimensions, ordered as the newcomer's journey: discoverability & first impression, the contribution contract, community health & governance, time to first build, task surface, code orientation for feature work, guidelines as an executable spec (the heart of the audit), self-verification before the push, fork-safe CI & merge gates, and the review & release loop measured from git/GitHub history
- A four-level grade per check — 🔴 Barrier / 🟡 Friction / 🟢 Clear / ⚪ Unverifiable from code — rolled up to a mechanical verdict: Closed to contributors / Open with friction / Contributor-ready. Task surface and the review & release loop are never graded 🔴 — they cap invitation and stewardship, never the verdict, and a 🟡 review-loop finding always leads the report with the real numbers instead of being buried
- A closed, exhaustive list of the ~22 conditions that may ever be graded 🔴 — nothing outside it blocks the verdict
- A **Contributor Readiness Plan**: every barrier and friction point as an ordered, P0/P1/P2-prioritized row naming the concrete artifact to produce, its effort (S/M/L), and its evidence — mapping 1:1 to graded checks, no padding
- A **first contribution dry run**: an eight-stage trace (land → decide it's alive → set up → pick something → find where the code goes → meet the standards → prove it works → open the PR) derived entirely from grades already assigned, naming the exact stage where a newcomer's path would stop
- Safe execution rules: check-only format/lint/typecheck and the test command may be run once, non-interactively, to measure exit codes and runtime; setup, install, migrate, deploy, and any fork/clone/branch/push/comment/label action are never run
- The report is printed in chat **and** saved to a dated file under `docs/open-source-audits/` in the audited project — never overwritten, so a maintainer can diff readiness over time — the one write the skill performs; it never commits, pushes, or creates the files it recommends
- Copyable starters for the files the Plan most often recommends: `CONTRIBUTING.md`, a PR template, two issue forms, `SECURITY.md`
- Configurable audience (`public` vs. `internal`) so an inner-source repo isn't graded against public-OSS expectations it was never meant to meet
- A crisp, one-paragraph boundary with `loop-engineering-audit` — that skill asks whether an unattended *agent* can develop the repo; this one asks whether an outside *human* can land a quality feature. Never run both from one report

## Layout

```
SKILL.md                                  # hub: discovery, the ten dimensions, grades, gotchas (agent entrypoint)
references/
  dimensions.md                           # full checklist per dimension, grade ladder, exhaustive 🔴 list, caps
  audit-workflow.md                       # phase-by-phase procedure, allowed vs. forbidden commands, dated reports
  output-format.md                        # the report markdown: plan, dry run, findings, footer
  first-contribution.md                   # the eight-stage dry run: what it inspects, what it may run, honesty rules
  config.md                               # .eagerworks/audit-open-source-repo.json schema
assets/
  readiness-report.md                     # report template → docs/open-source-audits/YYYY-MM-DD-<slug>.md
  audit-open-source-repo.example.json     # starter config
  CONTRIBUTING.template.md                # starter for Plan rows closing checks 2.1, 2.2, 2.4-2.7
  PULL_REQUEST_TEMPLATE.md                # starter for check 5.4
  ISSUE_TEMPLATE/bug_report.yml           # starter for checks 5.1-5.2
  ISSUE_TEMPLATE/feature_request.yml
  SECURITY.template.md                    # starter for check 3.2
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration required. To declare the audience (`public` vs. `internal`), a CLA/DCO stance, pin the standards commands, change budgets or history sample sizes, cap the plan, turn off the dry run, or disable a dimension that doesn't apply, add `.eagerworks/audit-open-source-repo.json` — see [`references/config.md`](references/config.md) and [`assets/audit-open-source-repo.example.json`](assets/audit-open-source-repo.example.json). Every dimension excluded by audience, every disabled dimension, and every non-default budget or cap that changed a grade is always disclosed in the report footer.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill audit-open-source-repo
```
