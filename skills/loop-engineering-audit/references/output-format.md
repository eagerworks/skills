# Loop Engineering Audit — Output Format

The same markdown is printed in chat and saved to `docs/loop-engineering-audit.md`. Fill `assets/audit-report.md`; every section below is required, in this order.

## 1. Header

```markdown
# Loop Engineering Audit — <repo name>

- **Date:** YYYY-MM-DD
- **Commit:** `<short sha>` on `<branch>`
- **Stack:** Rails 7.1 / Node 20 + TypeScript (pnpm workspace: `apps/web`, `packages/api`)
- **Verdict:** 🔴 Not ready | 🟡 Partially ready | 🟢 Ready
- **Blockers:** N · **Gaps:** N · **Unverifiable:** N
```

Then one paragraph — three sentences at most — saying what the loop would trip over first and what the biggest single win is.

## 2. Scorecard

One row per dimension, worst check wins:

```markdown
| # | Dimension | Grade | Worst check | Evidence |
|---|---|---|---|---|
| 1 | Agent-facing context | 🔴 | 1.1 no `AGENTS.md`/`CLAUDE.md` | `ls` at root |
| 2 | Reproducible environment | 🟢 | — | `bin/setup:1-40`, `.tool-versions` |
| 3 | Fast deterministic verification | 🟡 | 3.5 suite 8m41s | `bundle exec rspec` exit 0, 8m41s |
...
```

## 3. Work Plan

**This is the deliverable** — the ordered list of work required before the project can be developed with loop engineering. Blockers first, then gaps by leverage (`references/audit-workflow.md` → Phase 4).

```markdown
| # | Dim | Grade | Task | Effort | Evidence |
|---|---|---|---|---|---|
| 1 | 1 | 🔴 | Add `AGENTS.md` at the root stating stack, `bin/lint`, `bin/test`, and the "never run `db:reset`" rule — start from `assets/AGENTS.example.md` | S | no agent instructions file |
| 2 | 3 | 🔴 | Add `"test": "vitest run"` to `package.json`; the current `"test"` is `echo "no tests"` | S | `package.json:12` |
| 3 | 3 | 🟡 | Tag slow specs and document `bin/rspec-fast` running the unit subset (< 2 min) | M | full suite 8m41s |
```

Effort: **S** < 1 h · **M** ~ half a day · **L** more than a day.

## 4. Automation map

Where the delivery workflow stands today — an inventory, not a grade. Statuses are the five words in `references/loop-catalog.md`: **Automated** · **Assisted** · **Manual** · **Human by design** · **Absent**. They never appear in the Scorecard and never touch the verdict.

All ten stages are always listed, including the ones that are Absent — "there is no release process here" is information.

```markdown
| # | Stage | Status | Evidence | Loop candidate |
|---|---|---|---|---|
| 1 | Task intake & refinement | Manual | no `.github/ISSUE_TEMPLATE/`; none of the last 10 issues carry acceptance criteria | Backlog refinement |
| 2 | Implementation | Manual | no agent config beyond `CLAUDE.md` | Issue → PR |
| 3 | Local verification | Assisted | `bin/rubocop`, `bin/rspec` exist; nothing runs them pre-commit (no `.husky/`, no `lefthook.yml`) | — |
| 4 | Code review | Manual | no `CODEOWNERS`, no review bot | Agent PR review |
| 5 | Merge | Human by design | 1 approval required on `main` (`gh api .../protection`) | — |
| 6 | Release / changelog | Absent | no tags (`gh release list` empty), no `CHANGELOG.md` | — |
| 7 | Deploy | Automated | `.github/workflows/deploy.yml` on `push: [main]` | — |
| 8 | Post-deploy verification | Manual | no smoke-test step in `deploy.yml`, no error-tracker config | — |
| 9 | Dependency & security maintenance | Manual | no `dependabot.yml`/`renovate.json`; `bundler-audit` not in CI | Dependency & advisory patch |
| 10 | Repo hygiene | Manual | 14 `xit` in `spec/services` (`grep -rn xit spec/services`); `AGENTS.md` names 2 commands that don't exist | Docs drift-sync |
```

`Loop candidate` holds a loop name from `references/loop-catalog.md` or `—`. A stage graded **Human by design** always gets `—`: the map never proposes automating away a human decision.

## 5. Recommended loops

```markdown
_Advisory — not part of the Work Plan, not counted in the verdict._
```

That subtitle is required, verbatim, every time the section is printed.

At most five loops, ordered by prerequisites-met, then risk level, then evidence strength (`references/loop-catalog.md` → Selection rules). The index table first:

```markdown
| # | Loop | Risk | Why here (evidence) | Blocked by |
|---|---|---|---|---|
| L1 | Docs & `AGENTS.md` drift-sync | L1 Contained | 1.6 🔴 — `AGENTS.md` names `bin/test` and `bin/lint`; neither exists in `bin/` | #1 |
| L2 | Dependency & advisory patch | L1 Contained | `Gemfile.lock` committed (2.2 🟢), no `dependabot.yml`, 23 gems behind latest | — |
| L3 | Test-backfill | L1 Contained | 4.2 🟡 — `app/services` has 18 classes, `spec/services` has 4 | #4 |
```

When the concurrency floor is unmet (dimension 8 has 🟡s), add one line under the index table naming the loops and the Work Plan items:

```markdown
_Run these sequentially — 8.3 and 8.4 are 🟡, so two sessions would share a test database and a port (Work Plan #5, #6)._
```

Then one block per loop, in the same order — five short lines, no prose paragraphs:

```markdown
**L2 · Dependency & advisory patch loop** — L1 Contained

- **Trigger:** weekly cron; plus the advisory path when `bundler-audit` fails in CI.
- **Unit → done:** one gem per PR (majors alone); done when `bundle install && bin/rubocop && bin/rspec` exit 0 in CI and the PR states the upstream change.
- **Guardrails:** one dependency per PR · majors never auto-merged · max 5 open bot PRs · the advisory path runs the same suite as the routine path.
- **Configure:** `.github/dependabot.yml` (weekly, grouped, `open-pull-requests-limit: 5`); add `bundler-audit` as a CI step so the advisory trigger is real; keep `main` protected so auto-merge still waits for checks.
- **Maintain:** one named owner · weekly queue triage · kill switch: pause the schedule in `dependabot.yml`.
```

**Blocked by** holds the Work Plan item numbers whose checks are this loop's unmet prerequisites, or `—`. This is the only link between section 3 and section 5, and it points one way: a loop names the Work Plan items that unblock it, and the Work Plan never gains a row because a loop wants one.

When no loop is viable, print the section with the subtitle and one sentence — never an empty table and never a silent omission:

```markdown
## 5. Recommended loops

_Advisory — not part of the Work Plan, not counted in the verdict._

No loop is viable yet: 3.1 and 6.1 are 🔴, so nothing here has a check to verify with or a finish line to detect. The first loop this repo unlocks is the **docs & `AGENTS.md` drift-sync loop** (L1 Contained), viable once Work Plan items #1–#3 are done.
```

## 6. Findings by dimension

For each dimension (all eight), every non-🟢 check with its evidence and the concrete fix; 🟢 checks as a single compact line (`🟢 2.1, 2.2, 2.5, 2.7`). Keep each finding to: grade, check id, one-sentence problem, evidence, fix.

```markdown
### 3. Fast deterministic verification — 🟡

- 🟡 **3.5 Unit suite over budget** — `bundle exec rspec` took 8m41s (budget 5m). Fix: tag `type: :system` specs and add a documented fast subset.
- 🟡 **3.6 Skipped tests accumulating** — 14 `xit` in `spec/services/` (`grep -rn "xit" spec/services`). Fix: delete or fix each; add a lint rule against `xit`.
- 🟢 3.1, 3.2, 3.3, 3.4, 3.7
```

```markdown
### 8. Parallel-session readiness — 🟡

- 🟡 **8.3 Shared test database** — `config/database.yml` hardcodes `database: app_test` for the `test` environment. Fix: derive it from an env var with a per-worktree default (`ENV.fetch("TEST_DB") { "app_test_#{File.basename(Rails.root)}" }`).
- 🟡 **8.1 `.env` not reproduced in a new worktree** — no `.env.example` and `bin/setup` doesn't generate one. Fix: add `.env.example` and a `cp -n .env.example .env` step in `bin/setup`.
- 🟢 8.2, 8.5, 8.6, 8.7
```

## 7. Unverifiable from code

Every ⚪, each with the exact question or setting a human must confirm:

```markdown
- ⚪ **6.4 Branch protection** — `gh api .../protection` returned 403. Ask: are `lint` and `test` required status checks on `main`?
```

## 8. Footer

```markdown
---
_Generated by the `loop-engineering-audit` skill · dimensions disabled by config: none · loops: 3 recommended (advisory), 1 candidate filtered by `loops.maxRiskLevel: 1` · commands executed: `npx vitest run`, `npx tsc --noEmit`_
```

The footer must list every command the audit actually **ran** (not merely read), any dimension disabled via config, and how many loops were recommended and how many candidates were removed by `loops.maxRecommended` or `loops.maxRiskLevel` — or `loops: disabled by config` when `loops.enabled` is `false` — so the reader knows the audit's blast radius and blind spots.

## Rules

- Verdict is mechanical: any 🔴 → Not ready; else any 🟡 → Partially ready; else Ready. ⚪ never moves it.
- Work Plan rows map 1:1 to 🔴/🟡 checks — no extra rows.
- No finding without evidence (`references/rubric.md` → Conservatism rule).
- Chat output is the full report, then a one-line note: `Saved to docs/loop-engineering-audit.md (unstaged).`
- Sections 4 and 5 are **advisory**: they add no Work Plan rows, change no check or dimension grade, and never move the verdict or the Blockers / Gaps / Unverifiable counts. A repo can be 🟢 Ready with five recommended loops, or 🔴 Not ready with none.
- The automation map's statuses are the five plain words from `references/loop-catalog.md` — never 🔴/🟡/🟢/⚪, never a Scorecard row, never summed into anything.
- A recommended loop whose prerequisites aren't 🟢 yet is still listed, with the Work Plan item numbers that unblock it under **Blocked by**. That citation is the only coupling between sections 3 and 5, and it runs 5 → 3 only.
- At most 5 recommended loops (`loops.maxRecommended`); every candidate removed by that cap or by `loops.maxRiskLevel` is disclosed by count in the footer, never dropped silently.
- No recommended loop without evidence, and none the repo can't run (`references/loop-catalog.md` → Selection rules 4–5). Zero recommendations is a valid result, printed as the one-line "first loop unlocked" answer — not an empty table.
- With `loops.enabled: false` (`references/config.md`), sections 4 and 5 are omitted entirely and the footer reads `loops: disabled by config` — same non-silent-skip rule as a disabled dimension.
- No dimension-8 check is ever 🔴, and the audit never creates a worktree to grade one (`git worktree add` would be a second write) — anything needing a live worktree is ⚪ with the question.
