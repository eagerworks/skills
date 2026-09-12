# Loop Engineering Audit — Workflow

End-to-end procedure. Read `references/rubric.md` for what each check means; this file is about *how* to run the audit safely and what to do with the result.

## Phase 0 — Config

Look for `.eagerworks/loop-engineering-audit.json` at the repo root (`references/config.md`). It can move the report, disable dimensions, and change time budgets. Note anything it changes — it goes in the report's disclosure line.

## Phase 1 — Discovery

1. Detect stack(s) and workspaces (`SKILL.md` → Discovery).
2. Read the agent-facing surface in full: `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/*`, `.github/copilot-instructions.md`, `README.md`, `CONTRIBUTING.md`, `.claude/settings*.json`, `.devcontainer/*`, `.github/workflows/*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/ISSUE_TEMPLATE/*`, `docker-compose*.yml`, `Makefile`/`justfile`/`Taskfile.yml`.
3. Build the **command inventory**: every lint/typecheck/test/build/setup command from `package.json#scripts`, `turbo.json`, `Rakefile`/`lib/tasks`, `bin/*`, `pyproject.toml`, `Makefile`, and every `run:` step in CI. For each record: source, documented where, interactive?, exit-code correct?, runtime (Phase 2).

```bash
git rev-parse --show-toplevel
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null   # default branch
ls -a; ls bin 2>/dev/null; ls .github/workflows 2>/dev/null
jq '.scripts' package.json 2>/dev/null
grep -n "^[a-zA-Z_-]*:" Makefile 2>/dev/null
bundle exec rake -T 2>/dev/null | head -50
```

4. Build the **workflow inventory** for the automation map (`references/loop-catalog.md`) and probe dimension 8 (parallel-session readiness, `references/rubric.md`) — both read-only, no execution:

```bash
# Automation map: what already does each of the ten delivery stages, or nothing does
ls .github/workflows .github/ISSUE_TEMPLATE 2>/dev/null
ls .github/dependabot.yml renovate.json .mergify.yml CODEOWNERS .github/CODEOWNERS Dangerfile 2>/dev/null
ls .husky lefthook.yml .pre-commit-config.yaml .overcommit.yml 2>/dev/null
ls CHANGELOG.md .changeset .release-please-manifest.json release-please-config.json 2>/dev/null
ls config/deploy.yml fly.toml vercel.json Procfile render.yaml 2>/dev/null
ls .claude/settings.json .claude/agents docs/loops 2>/dev/null
grep -l "on:" .github/workflows/*.yml 2>/dev/null | head -20
grep -rn "audit\|bundler-audit\|pip-audit\|npm audit" .github/workflows 2>/dev/null

# Dimension 8: worktree readiness
git worktree list
git config core.hooksPath
grep -rnE "/Users/|/home/[a-z]+/|\$HOME/" bin Makefile .github config 2>/dev/null | head -20
grep -rnE "container_name:|^ *ports:" docker-compose*.yml 2>/dev/null
grep -nE "database:|DATABASE_URL|REDIS_URL" config/database.yml .env.example 2>/dev/null
cat .gitignore
```

`docs/loops/*` (or an equivalent) is the signal that this project already runs loops — read every playbook it contains; those stages are **Automated**, and a recommendation that duplicates one is padding.

## Phase 2 — Verify the commands (the only execution step)

You may **run** a command only if all of these hold:

- It is lint, typecheck, unit test, or build — never setup, migrate, seed, deploy, publish, release, or anything touching `docker`, `db:`, `prisma migrate`, `--force`, `rm`.
- You have read its definition and it doesn't shell out to any of the above.
- It has a non-interactive form (`CI=1`, `--ci`, `vitest run`, `--no-watch`, `-q`).
- Dependencies are already installed (`node_modules/`, `vendor/bundle`, `.venv`) — **do not install them**; if they're missing, grade from source and mark runtime ⚪.

Run each allowed command once, non-interactively, with a timeout and a wall clock:

```bash
# ✅ correct
CI=1 timeout 900 bash -c 'time npx vitest run' 2>&1 | tail -20
CI=1 timeout 900 bash -c 'time bundle exec rspec --format progress' 2>&1 | tail -20
timeout 300 bash -c 'time npx tsc --noEmit' 2>&1 | tail -5
echo "exit: $?"

# ❌ wrong
bin/setup                 # may create DBs, install tools, prompt
npx jest                  # watch mode when a TTY is attached
bundle exec rails db:prepare
docker compose up
```

Record exit code and runtime. A command that hangs past its timeout is 🔴 on 3.3 with the timeout as evidence. If a test run fails because a service isn't running (Postgres, Redis), that's evidence for 2.6 — note it and move on, don't start the service.

Also run the read-only `gh` probes if `gh auth status` succeeds; every failure becomes ⚪ with the question for a human:

```bash
gh auth status
gh run list -b "$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's|origin/||')" -L 5
gh api "repos/{owner}/{repo}/branches/main/protection" 2>&1 | head -5
gh pr list -L 10 --state merged --json title,body
gh issue list -L 10 --json title,body,labels
gh label list -L 50
gh release list -L 5
gh api "repos/{owner}/{repo}/environments" --jq '.environments[].name' 2>&1 | head -5
gh api "repos/{owner}/{repo}/actions/workflows" --jq '.workflows[] | "\(.name) \(.state)"' 2>&1 | head -20
```

The last three are for the automation map (`references/loop-catalog.md`) — GET only, like the rest. A `403`/`404` makes the corresponding map row's evidence "unreadable via `gh`"; it never becomes a ⚪ check, because the map isn't graded.

## Phase 3 — Grade

Walk `references/rubric.md` dimension by dimension. For each check write the grade **and its evidence** as you go — a `file:line`, a command + exit code + runtime, or the documented claim that proved false. Roll each dimension up to its worst check. Disabled dimensions get one line: `_Dimension N disabled by config_`.

## Phase 4 — Write the Work Plan

Turn every 🔴 and 🟡 into a task, ordered:

1. All 🔴, ordered so that earlier items unblock later ones (setup before tests, tests before CI).
2. All 🟡, ordered by leverage — dimension 1 and 3 gaps first (they pay off on every loop iteration), then 2, 6, 7, **8**, 4, 5.

Each task: `#`, dimension, grade, the concrete change (one sentence, imperative, naming the file to create/edit), effort **S** (< 1 h) / **M** (half a day) / **L** (more), evidence. Point at `assets/AGENTS.example.md` for dimension 1 blockers. Don't invent tasks that don't trace to a graded check.

## Phase 5 — Automation map and loop recommendations

**Runs no commands.** Everything here is built from the Phase 1 workflow inventory, the Phase 2 read-only `gh` probes, and the Phase 3 grades. Phase 2 remains the only execution step in the audit — if you find yourself wanting to run something here, you don't have enough evidence for the recommendation and shouldn't make it.

1. **Classify all ten stages** with one of the five statuses and its evidence (`references/loop-catalog.md` → The automation map). Never leave a row blank; **Absent** is an answer. Mark a stage the repo deliberately keeps with a person as **Human by design**, with the artifact that proves it (required reviewers, a QA column, a protected environment).
2. **Check the floor.** If any write-floor check (1.1, 1.3, 1.6, 2.2, 2.3, 2.4, 2.7, 3.1, 3.3, 3.4, 6.1) is 🔴, recommend **zero** loops: name the single loop the repo unlocks first and the Work Plan item numbers that unblock it, and stop at step 5.
3. **Select.** Gather every catalog loop whose "recommend when" signals are actually present with citable evidence; drop anything above `loops.maxRiskLevel`; drop anything on the never-recommend list; sort by the four-key ordering rule; truncate to `loops.maxRecommended`. Count what the cap and the risk filter removed — the footer discloses both.
4. **Fill each entry**: trigger, unit → done signal, guardrails, what to configure, and the owner/cadence/kill-switch line. Set **Blocked by** to the Work Plan item numbers whose checks are this loop's unmet prerequisites, or `—`. If the concurrency floor is unmet, add the sequential-run line naming the dimension-8 Work Plan items.
5. **Do not touch the Work Plan, the grades, the counts, or the verdict.** Not one row, not one grade. A loop that needs work adds nothing to the plan — it cites the plan.

Recommending nothing is a normal outcome, and for a repo with 🔴s it's the *correct* outcome. Never soften it by listing loops "for later" beyond the single first-unlocked one.

## Phase 6 — Deliver: chat first, then the file

1. Fill `assets/audit-report.md` (format: `references/output-format.md`).
2. **Print the complete report in chat** — the whole thing, not a summary.
3. Save the identical markdown:

```bash
mkdir -p docs
# write the report to docs/loop-engineering-audit.md (or reportPath from config)
```

Overwrite an existing report; the project keeps one current audit. Do **not** `git add`, commit, or push it — tell the user it's there and unstaged. If the project's `.gitignore` ignores `docs/`, say so instead of working around it.

## Re-auditing

When asked to re-check after work was done, run the full audit again (don't diff the old report — a fix can regress another check) and, if a previous report exists, add a short **Since last audit** section listing checks that changed grade.

If the repo has `docs/loops/*` playbooks (or equivalent) that weren't there last time, say so: those stages moved to **Automated**, the corresponding recommendation is retired, and "Since last audit" notes which loops were adopted alongside which checks changed grade.
