# Loop Engineering Audit — Loop Catalog

The single source of truth for the **automation map**, the **loop risk ladder**, the **loop catalog**, and the rules for selecting from it. `SKILL.md` only summarizes this page.

Everything here is **advisory**. Nothing on this page produces a Work Plan row, changes a dimension grade, or moves the verdict (`references/output-format.md` → Rules). Check ids (`1.6`, `3.5`, `8.3`) are cited from `references/rubric.md`; this page never redefines one and never adds one.

## The automation map

Inventory the delivery workflow before recommending anything. Every stage gets one status and its evidence — including the stages that should stay human.

Statuses are **plain words on purpose**: they measure who does the work today, not readiness, and must never be read as a fifth grade alongside 🔴/🟡/🟢/⚪.

| Status | Means | Evidence looks like |
|---|---|---|
| **Automated** | A machine does the whole stage unattended; no human step on the normal path | a workflow with a matching `on:` trigger, a bot config, a scheduled job |
| **Assisted** | Tooling exists but a human starts or finishes every run | a script in `bin/`, Dependabot PRs someone triages, a hand-written `CHANGELOG.md` |
| **Manual** | A person does it from scratch each time | nothing in the repo does it, but the stage clearly happens |
| **Human by design** | Deliberately a person's call, and should stay one | `CODEOWNERS` + required reviewers, a deploy environment with required approvers, a QA column on the board |
| **Absent** | The stage doesn't exist in this project at all | no deploy config anywhere, no tags or releases ever |

Classify all ten stages, with evidence, even when the answer is **Absent**:

| # | Stage | Read these |
|---|---|---|
| 1 | Task intake & refinement | `.github/ISSUE_TEMPLATE/*`, `gh issue list`, `gh label list`, a board |
| 2 | Implementation | `.claude/agents/*`, `.cursor/`, any agent workflow in `.github/workflows/*` |
| 3 | Local verification | `.husky/`, `lefthook.yml`, `.pre-commit-config.yaml`, `.overcommit.yml`, `bin/*` |
| 4 | Code review | `CODEOWNERS`, `Dangerfile`, `.github/auto-assign.yml`, a review bot workflow |
| 5 | Merge | branch protection (`gh api .../protection`), `.mergify.yml`, auto-merge settings |
| 6 | Release / changelog | `gh release list`, `CHANGELOG.md`, `.changeset/`, `release-please`/`semantic-release`/`release-drafter` config |
| 7 | Deploy | `.github/workflows/deploy*.yml`, `config/deploy.yml`, `fly.toml`, `vercel.json`, `Procfile`, k8s manifests, `gh api .../environments` |
| 8 | Post-deploy verification | a smoke/health-check step, an error-tracker or uptime config, a rollback script |
| 9 | Dependency & security maintenance | `.github/dependabot.yml`, `renovate.json`, `bundler-audit`/`npm audit`/`pip-audit` in CI |
| 10 | Repo hygiene (flaky tests, doc drift, lint debt) | `xit`/`.skip`/`.only` counts, `retry` wrappers, lint ignore lists, this audit's own dimension-1 accuracy findings |

### Stages that should stay human

**Automated is not the target state for every row.** A repo with real users whose whole column reads Automated is a finding a human should question, not a goal. Grade these **Human by design** when a person is doing them, and never propose a loop that removes them:

- **Merge authorization on the production branch** — someone accountable presses merge.
- **Deploy sign-off to production**, and the decision to roll back.
- **Exploratory and manual QA** — the tests nobody wrote yet are the point.
- **Security exceptions, credential rotation, and access grants.**
- **Product prioritization** — what to build, and whether to build it at all.

An agent loop can *prepare* every one of these — a PR ready to merge, a deploy waiting at an approval gate, a QA checklist drafted from the acceptance criteria. It doesn't get the last click.

## The risk ladder

Levels are defined by **blast radius and reversibility** — what the loop can damage and how hard it is to undo. Never by how likely the agent is to be wrong: a careful agent on an L3 loop is still an L3 loop, and blast radius is a property of the configuration, which the repo can actually prove.

| Level | Name | The loop can… | Undo | The guardrail that makes it acceptable |
|---|---|---|---|---|
| **L1** | **Contained** | write only to a branch it created, a PR, a PR/issue comment, an issue it opened, a label | close the PR; nothing landed | human merge is mandatory — branch protection on the default branch, the loop holds no merge permission, no `--admin`, no auto-merge |
| **L2** | **Shared-state** | change what the team reads as truth without anyone accepting it — commit to the default branch, enable auto-merge, edit or close issues, move a tag | a revert commit, but everyone else already pulled it | required status checks the loop cannot bypass **and** a path allowlist **and** a batch cap **and** a named owner notified on every run **and** a kill switch |
| **L3** | **External-effect** | reach outside the repo — deploy, publish a package, migrate real data, post into a third-party system | a rollback procedure, sometimes nothing | a human approval gate the loop cannot pass by itself (a protected environment with required reviewers), the loop never holds the production credential, and a rollback command that has actually been run |

**Default posture.** Recommend L1 freely when the prerequisites hold. Recommend L2 only with every guardrail in its row either already present or listed under "what to configure". Recommend **L3 only when the repo already has an automated deploy, a rollback path, and a readable post-deploy signal** — otherwise the stage stays `Human by design` and the loop is not recommended. Config can lower the ceiling (`loops.maxRiskLevel`, default `2` — `references/config.md`); it can never raise viability.

## Prerequisite floors

Per-loop prerequisites below are **in addition to** one of these floors. A floor check that isn't 🟢 doesn't disqualify the loop — it moves the loop's **Blocked by** column to the Work Plan items that fix it.

- **Write floor** — any loop that pushes code: **1.1, 1.3, 1.6, 2.2, 2.3, 2.4, 2.7, 3.1, 3.3, 3.4, 6.1** all 🟢. Without these the loop has no instructions it can trust, no clean clone, no non-interactive check, and no finish line.
- **Read floor** — loops that only read the repo and write comments or issues: **1.1, 1.2, 1.4** 🟢.
- **Concurrency floor** — running two or more loops *at the same time*: dimension 8 clean, especially **8.1, 8.3, 8.4**. Unmet, the loops are still recommended; the report says to run them sequentially.

## The catalog

Ten loops. Each entry: what it does · trigger · unit of work → done signal · recommend when · prerequisites · risk · guardrails · configure · maintain.

### 1. Issue → PR implementation loop — **L1 Contained**

- **What it does:** takes a labeled, well-formed issue, implements it on a branch, runs the project's own checks until they pass, opens a PR, and stops.
- **Trigger:** an `agent-ready` label added to an issue; plus `workflow_dispatch` with an issue number.
- **Unit → done:** one issue. Done when lint, typecheck, test and build all exit 0 on the branch **and** a PR is open against the default branch linking the issue. The agent's own opinion that it finished is not a done signal.
- **Recommend when:** 5.1 and 5.6 are 🟢 (issues really do carry acceptance criteria — sample them, don't trust the template), 3.1–3.4 🟢, and 4.2 🟢 for the area the labeled issues touch.
- **Prerequisites:** write floor + **5.1, 6.2, 4.2**. 6.2 matters more than it looks: if CI runs different commands than the docs name, a locally green agent branch goes red in CI and the loop has no finish line.
- **Risk:** L1 — its durable output is a branch and a PR, and a human merges.
- **Guardrails:** one issue per run; never merges; a cap of 3 verification cycles then stop and comment what failed; a path denylist (`db/migrate/**`, `db/schema.rb`, `.github/**`, `vendor/**`, generated dirs) that sends those changes to a human instead; its PRs get reviewed like anyone else's.
- **Configure:** the `agent-ready` label; `.github/workflows/agent-implement.yml` on `issues: [labeled]` with a `concurrency` group; an `AGENTS.md` "Definition of done" section; narrow agent permissions —
  ```jsonc
  // ✅ correct — narrow allow, explicit deny
  { "permissions": {
      "allow": ["Bash(bin/lint)", "Bash(bin/test:*)", "Bash(git commit:*)", "Bash(gh pr create:*)"],
      "deny":  ["Bash(git push --force:*)", "Bash(gh pr merge:*)", "Bash(bin/deploy:*)", "Bash(*db:reset*)", "Read(./.env)"] } }

  // ❌ wrong — a blanket allow means the loop has no risk level at all
  { "permissions": { "allow": ["Bash(*)"] } }
  ```
  plus branch protection on the default branch requiring the CI checks and one human approval.
- **Maintain:** owner = the tech lead; monthly review of merged-vs-closed PRs and median human edits; kill switch = drain the label or disable the workflow.

### 2. Backlog refinement loop — **L2 Shared-state**

- **What it does:** turns thin issues into well-formed tasks — restates the problem, drafts acceptance criteria, names the files likely involved, and flags what a human must decide.
- **Trigger:** a `needs-refinement` label; plus a weekly cron over the untriaged column.
- **Unit → done:** one issue. Done when the body carries an acceptance-criteria section and the loop has either applied `agent-ready` or posted the single question blocking it.
- **Recommend when:** 5.1 is 🔴/🟡, or 5.6 shows templates exist but aren't followed — and the tracker is readable (5.5/5.6 not ⚪).
- **Prerequisites:** read floor + **5.5** 🟢.
- **Risk:** L2 — it edits a shared artifact the team reads as truth, and nothing accepts the edit before it lands.
- **Guardrails:** append into a marked `## Proposed acceptance criteria (agent draft)` block, never rewrite a human's words; never close, reprioritize, or reassign; never invent acceptance criteria for a product question — post the question; cap issues per run.
- **Configure:** `.github/ISSUE_TEMPLATE/*` with the AC section the loop fills; both labels; a workflow on `issues: [labeled]` + `schedule:`; a token scoped to `issues: write` only — no `contents: write`.
- **Maintain:** owner = whoever runs planning; reviewed at every planning session; kill switch = remove the label.

### 3. Test-backfill loop — **L1 Contained**

- **What it does:** adds tests for an untested module, one module per PR, with no production-code change.
- **Trigger:** a campaign kickoff naming a target directory, or a cron that pops the next entry off a checked-in target list.
- **Unit → done:** one source file or module. Done when the new tests pass, that file's coverage increases, and the diff touches test files only.
- **Recommend when:** 4.1 🟢 but 4.2 🟡 — a framework exists and large areas are untested. The 4.2 finding's evidence *is* the queue.
- **Prerequisites:** write floor + **3.5, 3.7** 🟢 (a focused run, documented, inside budget); 4.5 helps by making the done signal measurable.
- **Risk:** L1.
- **Guardrails:** production code on the path denylist — a PR that touches it gets closed, not merged; assertion-free tests are rejected (check 4.3 is the bar); one module per PR so a bad batch is one revert; never deletes or skips an existing failing test to go green.
- **Configure:** a checked-in target list; a coverage tool with per-file output (`simplecov`, `c8`, `coverage.py`); the denylist in both the loop's prompt and `AGENTS.md`.
- **Maintain:** owner = the area owner; runs until the target list is empty, then **retires** — a finished campaign loop gets turned off, not left running.

### 4. Mechanical-refactor loop (lint/format/type debt, codemods) — **L1 Contained**

- **What it does:** applies one mechanical transform across the codebase in small, independently revertable batches — enabling a lint rule directory by directory, removing `any`, running a framework codemod.
- **Trigger:** a campaign kickoff naming the rule or codemod; or a weekly cron while the debt budget is non-zero.
- **Unit → done:** one directory or ≤N files. Done when the rule/codemod is clean for that batch and the full suite exits 0 **with no test file in the diff** — a behavior-preserving change that needed a test edit wasn't behavior-preserving.
- **Recommend when:** 3.1 was 🟡 and the gap is now closed, or the lint/typecheck command only passes thanks to a large ignore list — cite the count (`grep -rc "rubocop:disable" app`, `eslint-disable`, `: any`).
- **Prerequisites:** write floor + **3.2, 3.5, 4.2** 🟢. A mechanical change is only safe where a real suite covers the touched code.
- **Risk:** L1.
- **Guardrails:** one rule per campaign; a file cap per PR (~40) so review stays possible; the rule is promoted to *error* for each cleaned directory as its batch merges, so the debt can't regrow behind the loop.
- **Configure:** per-directory overrides in the lint/type config; a `.git-blame-ignore-revs` entry for format-only commits; the rule name and file cap written down with the campaign.
- **Maintain:** one owner per campaign, with the exit condition written down before the first run.

### 5. Dependency & advisory patch loop — **L1 Contained** (L2 with patch auto-merge; L3 if it publishes)

- **What it does:** opens one PR per dependency bump, runs the full suite, and summarizes the upstream change and its blast radius. Same machine, two triggers.
- **Trigger:** a weekly cron for routine bumps; an advisory signal for urgent ones (`bundler-audit`/`npm audit --audit-level=high`/`pip-audit` failing in CI, or a Dependabot alert).
- **Unit → done:** one dependency, majors always alone. Done when install + lint + typecheck + test + build exit 0 on a clean checkout and the PR states what changed upstream.
- **Recommend when:** 2.2 🟢 (a committed lockfile — without one this loop is meaningless), stage 9 of the map is Manual/Absent, and the manifest shows pins well behind.
- **Prerequisites:** write floor + **3.5, 4.2, 6.2, 6.5** 🟢. 6.5 is load-bearing: if the verify job needs secrets, the bot's PRs can't be verified and every one lands on a human's desk anyway.
- **Risk:** L1 as a PR-only loop. Auto-merging patch bumps makes it **L2**. A loop that also cuts a release is **L3** — split that out (see #10).
- **Guardrails:** one dependency per PR; majors never auto-merged; the advisory path runs the *same* verification as the routine path — an urgent fix that skips the suite is how a loop causes the outage it was fixing; a cap on open bot PRs (~5) so the queue can't flood review.
- **Configure:** `.github/dependabot.yml` or `renovate.json` with grouping and `open-pull-requests-limit` — let the platform bot do the bump and use the agent only for bumps that need a code change; add the audit command as a CI step so the advisory trigger is real; keep branch protection so auto-merge still waits for checks.
- **Maintain:** owner = whoever is on call for upgrades; weekly triage of the open queue; kill switch = pause the schedule in the bot config.

### 6. CI-failure triage loop — **L2 Shared-state**

- **What it does:** on a red build, reproduces the failure locally, classifies it (real regression / flake / infra / dependency), then either pushes a fix to the failing PR branch or opens an issue with the classification and the log excerpt.
- **Trigger:** `workflow_run: { types: [completed] }` filtered to `conclusion == 'failure'`.
- **Unit → done:** one failed run. Done when the same job is green again, or an issue exists naming the cause with the log lines.
- **Recommend when:** 6.1 🟢 and `gh run list` shows recent failures on the default branch (the 6.3 evidence), or 6.2 🟡 — local/CI divergence means humans are re-running builds by hand.
- **Prerequisites:** write floor + **6.2, 6.3** 🟢, and **3.6** 🟢 *or loop #7 first*. Triaging a flaky suite is how a loop learns that deleting tests makes builds green. Check **8.3** too: a shared test database makes concurrent runs look flaky when the real cause is collision.
- **Risk:** L2 — it pushes to branches it didn't create, and on a red default branch it proposes a fix to shared code; an infra failure misread as a code failure produces a confidently wrong fix.
- **Guardrails:** never force-pushes; **never edits `.github/**`** — that's the loop modifying its own gate, always a human PR; a failing test is fixed by changing production code, never by deleting or skipping it (quarantine is #7's job, with a record); one attempt per run, then escalate; any fix targeting the default branch goes through a PR with a required human approval.
- **Configure:** `.github/workflows/ci-triage.yml` on `workflow_run`; a `concurrency` group keyed by branch; `.github/**` in the deny list; a repo variable kill switch checked in the first `if:`.
- **Maintain:** owner = the CI owner; weekly check of what it classified and whether it was right; a loop whose classifications are wrong more than occasionally gets narrowed to issue-only.

### 7. Flaky-test quarantine loop — **L2 Shared-state**

- **What it does:** detects tests that fail then pass on re-run, tags them as quarantined with an owner and an expiry, and opens one issue per quarantined test.
- **Trigger:** a nightly cron running the suite N times; or a CI post-step that records reruns.
- **Unit → done:** one test. Done when it's tagged, excluded from the blocking run, and carries an issue with an expiry date.
- **Recommend when:** 3.6 🟡 **with evidence** — `retry` wrappers, `--only-failures` in CI, an accumulating `xit`/`.skip` count, or rerun evidence in `gh run list`. **Rule out 8.3 first:** a shared test database makes every parallel run look flaky, and quarantining those tests treats the symptom instead of the collision.
- **Prerequisites:** write floor + **3.7, 6.3** 🟢.
- **Risk:** L2 — it removes tests from the gate, i.e. it erodes the safety net every other loop depends on. That's the blast radius, not the code it writes.
- **Guardrails:** a hard cap on simultaneously quarantined tests (10, or 1% of the suite) — **hitting the cap stops the loop and escalates rather than quarantining more**; every quarantine carries an owner and an expiry; quarantined tests keep running in a second, non-blocking job so recovery is visible; a test covering auth, payments, or tenancy scoping is never auto-quarantined.
- **Configure:** a marker the runner understands (`:flaky` RSpec tag, `@pytest.mark.flaky`, a tagged skip with a lint rule); a non-blocking CI job for the quarantine set; a checked-in register of what is quarantined and until when.
- **Maintain:** owner reviews the register every sprint. A quarantine past its expiry is a bug in the loop, not an accepted state.

### 8. Agent PR-review loop — **L1 Contained**

- **What it does:** reviews every open PR against the repo's own written conventions and posts findings as comments. Never approves, never merges.
- **Trigger:** `pull_request: [opened, synchronize]`.
- **Unit → done:** one PR head SHA. Done when a review exists for that SHA.
- **Recommend when:** 1.4 🟢 — there are written conventions to review against; without them the loop invents a style guide and the team learns to ignore it. Plus review-load evidence: many open PRs, or 5.2 🟡 (no PR checklist).
- **Prerequisites:** read floor + **1.4, 1.5** 🟢.
- **Risk:** L1 — comments only. The real cost here is noise, not damage.
- **Guardrails:** never approves, never merges, never dismisses a human review; a cap on comments per PR; no comment without a `file:line` and a concrete failure; generated and vendored paths skipped; one review per head SHA so a push doesn't re-post the same findings.
- **Configure:** install the `pr-review` skill (or an equivalent) so the rubric is fixed rather than improvised, with its `ignorePaths` and suggestion cap set; a workflow token limited to `pull-requests: write`; `CODEOWNERS` so branch protection still requires a human review.
- **Maintain:** owner = whoever tunes the rubric; monthly check that comments are being acted on. An ignored review loop should be narrowed or switched off.

### 9. Docs & `AGENTS.md` drift-sync loop — **L1 Contained**

- **What it does:** checks that every command, path, and runtime claimed in `AGENTS.md`/`CLAUDE.md`/`README.md` still exists and still works, and opens a PR correcting the ones that don't.
- **Trigger:** `push` to the default branch touching `package.json`, `Gemfile`, `bin/**`, `Makefile`, `.github/workflows/**`; plus a monthly cron.
- **Unit → done:** one doc file. Done when every backticked command in it resolves and the commands the loop ran exit 0.
- **Recommend when:** **1.6, 1.7, or 8.7 was 🔴/🟡 in this audit.** The audit just proved drift happens here — that is the evidence, and it's the strongest justification in the catalog.
- **Prerequisites:** write floor minus 1.6 (that's what it fixes) + **3.2** 🟢.
- **Risk:** L1.
- **Guardrails:** docs-only path allowlist; it corrects the *claim* to match reality and never "fixes" reality to match the doc; it never rewrites conventions or the "never do this" list — a human owns policy, the loop owns commands, paths and runtimes; a documented command that is *missing* rather than wrong gets an issue, not a doc edit that quietly deletes the intent.
- **Configure:** a CI step that extracts backticked commands from the agent docs and asserts each exists (`references/rubric.md` → dimension 1 has the grep) — that check, not the loop, is what makes drift visible; the path allowlist; the cron.
- **Maintain:** owner = the agent-docs owner; reviewed whenever dimension 1 is re-graded; kill switch = disable the workflow.

### 10. Release-notes loop — **L2 Shared-state** (L3 if it publishes)

- **What it does:** drafts the changelog entry or release notes from the PRs merged since the last tag, grouped by area, in the repo's own format.
- **Trigger:** a tag push, a release branch cut, or `workflow_dispatch`.
- **Unit → done:** one release range. Done when a PR exists whose `CHANGELOG.md` entry (or draft GitHub Release) covers every merged PR in the range with none dropped.
- **Recommend when:** releases/tags exist, stage 6 of the map is Assisted/Manual, and **5.3 🟢** — without a commit or label convention to group by, the grouping is guesswork dressed as a summary.
- **Prerequisites:** read floor + **5.3, 5.4** 🟢 and a readable release history (`gh release list`).
- **Risk:** L2 as a changelog PR — it asserts what shipped. **L3 if it publishes the release or pushes the tag**, because publishing usually triggers a deploy. Keep publishing human.
- **Guardrails:** draft only — never `gh release create` without `--draft`, never pushes a tag; every entry traces to a merged PR number; a PR it can't classify goes under a "Needs a human" heading rather than being silently dropped; never edits past entries.
- **Configure:** the convention it groups by; `release-drafter`/`changesets`/`release-please` for the mechanics with the agent supplying prose; a stated `CHANGELOG.md` format; a protected environment for publishing so the loop structurally cannot.
- **Maintain:** owner = the release manager; reviewed each release; kill switch = `workflow_dispatch`-only.

### Deliberately not in this catalog

- **A deploy loop.** Deploying to production is `Human by design`. Its automatable parts — running the pipeline, reading its status, rolling back on a failed health check — belong to the deploy tooling behind an approval gate, not to an agent loop. A repo that genuinely wants one is in L3 territory and must meet every L3 condition first.
- **A "work the Work Plan" loop.** Most Work Plan items are policy decisions — what belongs in the never-run list, what the time budgets are, which dimension is worth disabling. That's why the plan is written for a human.

Lint/type debt and codemods are one entry (#4) because they have the same shape and differ only in trigger — a recurring drip versus a finite campaign. Dependency upgrades and security advisories are one entry (#5) with two triggers, for the same reason. Don't split them back apart.

## Selection rules

1. **Cap: five.** `loops.maxRecommended`, default 5. The Work Plan is the deliverable; a recommendation list longer than it inverts the report. Five is already a multi-quarter roadmap for most teams.
2. **Order deterministically** — sort by, in strict order:
   1. prerequisites fully 🟢 before loops that are blocked;
   2. lower risk level first (L1 → L2 → L3);
   3. evidence strength — a loop justified by a 🔴 finding in *this* audit outranks one justified by a 🟡, which outranks one justified by a neutral repo signal;
   4. fewer things to configure first.
3. **Recommend none** when any write-floor check is 🔴. Then print the one-line answer instead: the single loop this repo unlocks first, its risk level, and the Work Plan item numbers that unblock it. For a repo with no test command and no CI, "no loop is viable yet, and here is the first one" *is* the correct answer to "which loops should we run".
4. **Every recommendation cites evidence** — the rubric's conservatism rule applied to loops: a path, a command's output, a `gh` probe result, or a check id and its grade from this audit. A loop with no citable signal is not recommended no matter how good an idea it is.
5. **Never recommend** — the anti-padding list:
   - a deploy loop when stage 7 is **Absent**;
   - a release-notes loop in a repo with no tags and no releases;
   - a flaky-quarantine loop with no flake evidence;
   - a dependency loop with no committed lockfile (2.2 🔴);
   - a backlog-refinement loop when the tracker is ⚪;
   - any loop whose done signal the repo cannot produce — no CI means no loop has a finish line;
   - any loop that duplicates something already **Automated** in the map. Renovate already running means stage 9 is Automated: say so and move on.
6. **Concurrency is a rendering rule, not a filter.** When the concurrency floor is unmet, still recommend the loops and add one line under the index table: run them sequentially until the dimension-8 Work Plan items are done. A 🟡 on dimension 8 never removes a loop from the list.
7. **Disclose every removal.** Candidates dropped by `loops.maxRecommended` or filtered by `loops.maxRiskLevel` are reported by count in the footer. A cap is a skip, and skips are never silent.
8. **Never build a loop.** The audit recommends; it writes no workflow, no permissions file, no playbook. Its one write is still the report (`docs/decision-records/2026-08-28--audit-report-saved-to-docs.md`). Describe the change and point at the files to create.

## Keeping a loop alive

A loop nobody owns is a loop that will be quietly broken for a month, then loudly wrong. Every recommended loop's report block names four things:

- **Owner** — one person, not a team.
- **Review cadence** — monthly for L1, every sprint for L2, every run for L3. What gets reviewed: the last N outputs, accepted vs. reverted, and whether any guardrail fired.
- **Kill switch** — one lever that stops the loop with no code review: a repo variable checked in the workflow's first `if:`, draining the trigger label, or disabling the workflow. Named once, tested once.
- **Exit condition** — campaign loops (#3, #4) are finished, not eternal. Standing loops get one too: "retire if the merged-PR rate stays under 50% across two cadence checks".

Recommend the team record those four beside the loop's own workflow, or in their `docs/`. An existing loop-playbook directory (`docs/loops/*` or equivalent) is a signal those stages are already **Automated** — read it, and don't recommend a loop that duplicates one.

Worth tracking per loop: runs per week, PRs opened → merged → closed unmerged, median human edits before merge, guardrail trips, time to green. Three of those beat none.

## Running more than one loop at once

Concurrent loops multiply throughput only if the **concurrency floor** holds — dimension 8, especially a per-worktree database (8.3), non-colliding ports and containers (8.4), and a setup that regenerates the gitignored files a new worktree won't have (8.1). Two sessions sharing one test database don't run twice as fast; they produce two wrong answers. When dimension 8 has 🟡s, recommend the loops anyway and say they run **sequentially** until those Work Plan items are done.
