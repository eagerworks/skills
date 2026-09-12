# Audit Open Source Repo — Output Format

The same markdown is printed in chat and saved to `docs/open-source-audits/YYYY-MM-DD-<slug>.md`. Fill `assets/readiness-report.md`; every section below is required, in this order.

## 1. Header

```markdown
# Contributor Readiness Audit — <repo name>

- **Date:** YYYY-MM-DD
- **Commit:** `<short sha>` on `<branch>`
- **Stack:** Rails 7.1 / Node 20 + TypeScript
- **Audience:** public-oss | internal
- **Verdict:** 🔴 Closed to contributors | 🟡 Open with friction | 🟢 Contributor-ready
- **Barriers:** N · **Friction:** N · **Unverifiable:** N
```

Then a lead paragraph, three sentences max: where the path first breaks, and the single biggest win. **When any 10.x check is 🟡**, the first sentence must instead state the stewardship numbers (median time to first review, external PRs merged, oldest unanswered PR) — see `references/dimensions.md` → "Capped dimensions".

When a previous dated report exists under `docs/open-source-audits/`, add one line before the paragraph: `**Progress since <previous date>:** N checks improved, M regressed` (see `references/audit-workflow.md` → "Re-auditing").

## 2. Scorecard

One row per dimension, worst check wins:

```markdown
| # | Dimension | Grade | Worst check | Evidence |
|---|---|---|---|---|
| 1 | Discoverability & first impression | 🟢 | — | README.md:1-30, LICENSE (MIT) |
| 2 | The contribution contract | 🔴 | 2.1 no CONTRIBUTING.md | ls at root, .github/, docs/ |
...
```

## 3. Contributor Readiness Plan

**This is the deliverable** — the ordered, prioritized list of concrete artifacts needed before a capable outsider can land a quality feature. Items 1–N are P0: until they're done, a capable outsider cannot complete a first contribution.

```markdown
| # | P | Closes | Artifact | Change | Effort | Evidence |
|---|---|---|---|---|---|---|
| 1 | P0 | 2.1, 2.2, 2.5 | `CONTRIBUTING.md` | Write the fork → branch → PR flow, the commands from `make check`, and a one-sentence review-time expectation — start from `assets/CONTRIBUTING.template.md` | M | no `CONTRIBUTING*` at root, `.github/`, or `docs/` |
| 2 | P0 | 4.4 | `.env.example` | Add the 3 variables the app reads but the template omits: `S3_BUCKET`, `REDIS_URL`, `SENTRY_DSN` | S | `grep -rn "process\.env\." src \| sort -u` vs `.env.example` |
| 3 | P1 | 7.1, 7.5 | `package.json`, `CONTRIBUTING.md` | Add `"check": "npm run format:check && npm run lint && npm run typecheck && npm test"` and name it in CONTRIBUTING as the one command to run before pushing | S | `.prettierrc` exists; no check-only script; CI runs four separate steps |
```

Effort: **S** < 1 h · **M** ~ half a day · **L** more than a day.

**The hard rule — 1:1, no padding.** Every 🔴 and every 🟡 appears in the `Closes` cell of exactly one row, and every row cites at least one check id. A row may close several checks only when a single named artifact genuinely closes all of them (one `CONTRIBUTING.md` closes 2.1, 2.2 and 2.5; it does not also close 4.4). A row with no check id is padding — delete it however good the idea is. ⚪ checks produce no row: they produce a question in section 7.

**Priority ladder:**

| P | Meaning | Default assignment |
|---|---|---|
| **P0 — the door is shut** | A capable outsider cannot complete a first contribution at all | Exactly the 🔴s — P0 ⟺ 🔴, mechanically |
| **P1 — the feature won't meet the bar** | The contribution is possible, but the project can't hold it to its own standard without a reviewer teaching by hand | 🟡s in dimensions 6, 7, 8, 9 |
| **P2 — they won't find the work, or it won't reach users** | Discovery, invitation, governance polish, the release loop | 🟡s in dimensions 1, 3, 5, 10, and the expectation-setting 🟡s in 2 |

A 🟡 may be **raised** when the evidence warrants — most often when it blocks a P0 row's fix (you can't write the CONTRIBUTING command list before the aggregate command exists) — with the reason stated in the row. Never lowered.

**Ordering rule**, applied in strict order:

1. All P0, then all P1, then all P2.
2. Within P0, **earliest break in the journey first** — ascending by the lowest dimension number the row closes. Two P0s in the same dimension go in dependency order (`.env.example` before "the suite runs without secrets"; the linter config before the aggregate command; the aggregate command before CI invoking it).
3. Within P1, **leverage, enforcement first**: dimension 7 (a rule that runs teaches every contributor forever) before 8 and 6 (a rule in a doc teaches only the ones who read it) before 9.
4. Within P2, journey order again: 1 → 3 → 5 → 10.
5. Ties break by lower effort first, so one afternoon closes several rows.

**Cap.** `plan.maxItems`, default 25. P0 rows are **never** truncated — if P0s alone exceed the cap, the cap is ignored and the footer says so. Everything omitted still appears in *Findings by dimension*; the cap changes the plan, never a grade.

## 4. First contribution dry run

A view over grades already assigned — see `references/first-contribution.md` for the full spec. It adds no finding, no plan row, no grade, and moves no verdict.

```markdown
_A trace, not a simulation — every line is something the audit read, ran, or measured._

**First break: stage 3 (Set up).** `CONTRIBUTING.md:24` says `make bootstrap`; the `Makefile` has no `bootstrap` target (2.8 🔴). Plan item #2.

| # | Stage | Result | What was inspected | Evidence |
|---|---|---|---|---|
| 1 | Land | Clears | README.md:1-30, LICENSE (MIT), CONTRIBUTING linked at README.md:12 | 1.1-1.4 🟢 |
| 2 | Decide it's alive | Costs them | last commit 2026-09-02; last release v0.4.1, 2025-06-11 (15 months) | 10.8 🟡 |
| 3 | Set up | **Stops here** | `CONTRIBUTING.md:24` → `make bootstrap`; `Makefile` targets: `setup`, `test`, `lint` | 2.8 🔴 |
| 4 | Pick something | Costs them | `good first issue`: 0 open (label exists, `gh label list`) | 5.5 🟡 |
| 5 | Find where the code goes | Costs them | #318 "add CSV export" touched `src/routes/export.ts`, `src/services/export/`; no doc names `src/services/*` | 6.1, 6.2 🟡 |
| 6 | Meet the standards | Can't tell | `node_modules/` absent; the audit does not install | 7.7 ⚪ |
| 7 | Prove it works | Can't tell | same | 8.4 ⚪ |
| 8 | Open the PR | Costs them | `ci.yml` triggers on `pull_request` ✓; `test` job reads `secrets.CODECOV_TOKEN` (`ci.yml:34`) — empty on forks | 9.2 🔴 |
```

The **first-break sentence is required even when nothing breaks**: "No stage stops; the earliest cost is stage 4 (…)." Result words are plain text, never emoji, so nothing reads as a fifth grade: **Clears** / **Costs them** / **Stops here** / **Can't tell**.

## 5. Findings by dimension

For each dimension (all ten), every non-🟢 check with its evidence and the concrete fix; 🟢 checks as a single compact line.

```markdown
### 7. Guidelines as an executable spec — 🔴

- 🔴 **7.2 No linter or formatter** — no `.eslintrc*`/`eslint.config.*`, no `.prettierrc`, no `.editorconfig`. Fix: add ESLint + Prettier, commit the configs, add `lint`/`format:check` scripts.
- 🟡 **7.5 No aggregate command** — `package.json` has `lint`, `typecheck`, `test` as separate scripts, nothing runs all three. Fix: add a `check` script.
- 🟢 7.3, 7.4, 7.6, 7.7, 7.8

**Enforcement matrix**

| Standard | Config in repo | Command | Pre-commit | In CI | Green on default branch |
|---|---|---|---|---|---|
| format | — | — | — | — | — |
| lint | — | — | — | — | — |
| typecheck | `tsconfig.json:1` (`strict: true`) | `npm run typecheck` | ✗ | ✓ | exit 0, 6s |
| commit msg | — | — | — | — | — |
```

The **enforcement matrix** (dimension 7), the **fork-gate table** — `| Workflow | Trigger(s) | Uses secrets | Runs on fork PR | Required check | Median duration |` (dimension 9) — and the **stewardship metrics table** — `| Metric | Value | Sample |` covering 10.1–10.3 and 10.8 (dimension 10) — are mandatory whenever that dimension is graded, even when every check in it is 🟢.

## 6. Unverifiable from code

Every ⚪, each with the exact question or setting a human must confirm:

```markdown
- ⚪ **9.7 Branch protection** — `gh api .../protection` returned 403. Ask: are `lint` and `test` required status checks on the default branch?
```

## 7. Footer

```markdown
---
_Generated by the `audit-open-source-repo` skill · audience: internal (4 checks n/a: 1.3, 1.8, 3.1, 3.2) · dimensions disabled by config: none · commands executed: `npm run lint` (14s, exit 1), `npm test` (4m12s, exit 0) · budgets overridden: firstReviewDays 30 (default 14) · history sample: 12 of 20 PRs requested (repo has 12), `gh` authenticated · plan: 31 items, 6 omitted by plan.maxItems: 25 · dry run: included_
```

The footer must always carry: disabled dimensions; `runCommands: false` when set; every command actually **run** with time and exit code; the audience and the exact check ids it made n/a; the contribution-agreement stance when not `"none"`; every non-default budget that produced a 🟡, with the value used; the achieved history sample sizes and whether `gh` was readable (plus the count of ⚪s that caused); the plan cap and the omitted count; and `dry run: disabled by config` when off.

## Rules

- Verdict is mechanical: any 🔴 → 🔴 Closed to contributors; else any 🟡 → 🟡 Open with friction; else 🟢 Contributor-ready. ⚪ never moves it.
- No check outside `references/dimensions.md` → "The exhaustive 🔴 list" may ever be graded 🔴.
- No check in dimension 5 or 10 is ever 🔴 — see "Capped dimensions". When any 10.x is 🟡, the lead paragraph opens with the stewardship numbers, not the usual three-sentence summary.
- Contributor Readiness Plan rows map 1:1 to 🔴/🟡 checks — no extra rows, no row without a check id.
- No finding without evidence (`references/dimensions.md` → Conservatism rule); absence is only a finding when the project is the kind that needs the thing.
- Chat output is the full report, then a one-line note: `Saved to docs/open-source-audits/<filename> (unstaged).`
- The dry run (section 4) is a **view**: it adds no finding, no Plan row, changes no grade, and never moves the verdict. `dryRun.enabled: false` omits it and the footer says `dry run: disabled by config`.
- A dimension disabled by config gets one line in Findings: `_Dimension N disabled by config_`, and is excluded from the scorecard and the verdict, disclosed in the footer.
- `plan.maxItems` is a cap on the Plan only — P0 rows are never truncated by it, and every check it omits still appears in Findings by dimension.
