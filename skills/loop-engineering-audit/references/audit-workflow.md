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
```

## Phase 3 — Grade

Walk `references/rubric.md` dimension by dimension. For each check write the grade **and its evidence** as you go — a `file:line`, a command + exit code + runtime, or the documented claim that proved false. Roll each dimension up to its worst check. Disabled dimensions get one line: `_Dimension N disabled by config_`.

## Phase 4 — Write the Work Plan

Turn every 🔴 and 🟡 into a task, ordered:

1. All 🔴, ordered so that earlier items unblock later ones (setup before tests, tests before CI).
2. All 🟡, ordered by leverage — dimension 1 and 3 gaps first (they pay off on every loop iteration), then 2, 6, 7, 4, 5.

Each task: `#`, dimension, grade, the concrete change (one sentence, imperative, naming the file to create/edit), effort **S** (< 1 h) / **M** (half a day) / **L** (more), evidence. Point at `assets/AGENTS.example.md` for dimension 1 blockers. Don't invent tasks that don't trace to a graded check.

## Phase 5 — Deliver: chat first, then the file

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
