# Audit Open Source Repo — Workflow

End-to-end procedure. Read `references/dimensions.md` for what each check means; this file is about *how* to run the audit safely and what to do with the result.

## Phase 0 — Config

Look for `.eagerworks/audit-open-source-repo.json` at the repo root (`references/config.md`). It can move the report, set the audience and project type, disable dimensions, and change budgets and samples. Note anything it changes — it goes in the report's disclosure line.

## Phase 1 — Discovery

1. Settle the audience and project type (`SKILL.md` → Discovery, steps 1–2).
2. Read the contributor-facing surface in full: `README.md`, `LICENSE`, `CONTRIBUTING.md` (root, `.github/`, `docs/`), `CODE_OF_CONDUCT.md`, `SECURITY.md`, `SUPPORT.md`, `GOVERNANCE.md`, `CODEOWNERS`, `.github/ISSUE_TEMPLATE/*`, `.github/PULL_REQUEST_TEMPLATE*`, `.github/workflows/*`, linter/formatter/typecheck configs, `.editorconfig`, hook configs, `.env.example`, `docker-compose*.yml`, pinned-toolchain files, `CHANGELOG.md`, `docs/` (architecture, ADRs).
3. Build the **standards inventory** for dimension 7: every format/lint/typecheck command, its config file, whether a hook installs it, and whether an aggregate command exists.

```bash
git rev-parse --show-toplevel
git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null   # default branch
ls -a; ls .github/workflows .github/ISSUE_TEMPLATE 2>/dev/null
cat LICENSE 2>/dev/null | head -3
jq '.scripts, .license, .engines' package.json 2>/dev/null
ls .prettierrc* .eslintrc* eslint.config.* .rubocop.yml ruff.toml pyproject.toml .editorconfig 2>/dev/null
ls .husky lefthook.yml .pre-commit-config.yaml .overcommit.yml 2>/dev/null
```

4. Extract every backticked command, path, script and env var from `CONTRIBUTING.md` and the README's setup section, and resolve each against the tree (dimension 2, check 2.8; also feeds dry-run stage 3):

```bash
grep -oE '`[^`]+`' CONTRIBUTING.md README.md 2>/dev/null | sort -u
grep -rn "process\.env\.\|ENV\[\|os\.environ" --include='*.rb' --include='*.ts' --include='*.js' --include='*.py' . 2>/dev/null | sed -E 's/.*(process\.env\.[A-Z_]+|ENV\["?[A-Z_]+"?\]|os\.environ\["?[A-Z_]+"?\]).*/\1/' | sort -u
diff <(sort .env.example 2>/dev/null | cut -d= -f1) <(sort /tmp/found-vars 2>/dev/null)
```

5. Probe fork-safety for dimension 9, read-only:

```bash
grep -n "pull_request\b\|pull_request_target\|secrets\." .github/workflows/*.yml 2>/dev/null
grep -A3 "pull_request_target" .github/workflows/*.yml 2>/dev/null | grep -n "ref:\|head\."
```

## Phase 2 — Mine history

Every `gh`/`git` call is read-only (GET-equivalent); a `403`/`404` becomes ⚪ with the question, never a guess.

```bash
gh auth status
git log --oneline -30
git shortlog -sn --no-merges --since="12 months ago"
gh pr list --state merged -L "$(history.prSample)" --json number,author,createdAt,mergedAt,reviews,title
gh pr list --state open --json number,createdAt,updatedAt,comments
gh issue list -L "$(history.issueSample)" --json number,title,body,labels,createdAt,comments
gh label list -L 50
gh issue list --label "good first issue" --state open --json number,title,assignees,createdAt
gh release list -L 5
gh api repos/{owner}/{repo}/branches/<default>/protection 2>&1 | head -5
gh run list -b <default> -L 10
gh api repos/{owner}/{repo} --jq '.description, .topics' 2>&1
```

The most recently merged **feature** PR (not a docs/chore/dependency-bump PR) feeds dry-run stage 5: `gh pr view <number> --json files` lists the paths it touched.

## Phase 3 — Verify the standards commands (the only execution step)

You may **run** a command only if all of these hold:

- It is format-check, lint, typecheck, or the test command — never setup, migrate, seed, deploy, publish, release, or anything touching `docker`, `db:`, `--force`, `rm`.
- You have read its definition and it doesn't shell out to any of the above.
- It has a non-interactive form.
- Dependencies are already installed (`node_modules/`, `vendor/bundle`, `.venv`) — **do not install them**; if they're missing, grade from source and mark the runtime checks (7.7, 8.4, dry-run stages 6–7) ⚪.

Run each allowed command once, non-interactively, with a timeout and a wall clock:

```bash
# ✅ correct
timeout 300 bash -c 'time npm run lint' 2>&1 | tail -20
timeout 300 bash -c 'time npm run format:check' 2>&1 | tail -10
CI=1 timeout 900 bash -c 'time npm test' 2>&1 | tail -20
echo "exit: $?"

# ❌ wrong
npm install                # never install
bin/setup                  # may create DBs, install tools, prompt
npx jest                   # watch mode when a TTY is attached
docker compose up
```

Record exit code and runtime. Lint or format-check failing on an untouched checkout is 7.7 🔴 with the failing rule as evidence. A test failing because a service isn't running is evidence for 8.2 (or 4.6) — note it and move on, don't start the service.

## Phase 4 — Grade

Walk `references/dimensions.md` dimension by dimension. For each check write the grade **and its evidence** as you go — a `file:line`, a command with exit code and runtime, a count, or the documented claim that proved false. Roll each dimension up to its worst check. No check outside the exhaustive 🔴 list may ever be graded 🔴. Disabled dimensions get one line: `_Dimension N disabled by config_`.

## Phase 5 — Write the Contributor Readiness Plan

Turn every 🔴 and 🟡 into a row, following `references/output-format.md` → section 3's priority and ordering rules exactly (P0 ⟺ 🔴, journey order within P0, enforcement-first within P1). Point at the matching starter in `assets/` for each row that creates a file this skill ships a template for (`CONTRIBUTING.template.md`, `PULL_REQUEST_TEMPLATE.md`, the issue forms, `SECURITY.template.md`). Don't invent rows that don't trace to a graded check.

## Phase 6 — First contribution dry run

Build the eight-stage trace from grades already assigned — no new command, no new `gh` call beyond what Phase 1/2 already gathered for stage 5's feature PR. Full spec: `references/first-contribution.md`.

## Phase 7 — Deliver: chat first, then the file

1. Fill `assets/readiness-report.md` (format: `references/output-format.md`).
2. **Print the complete report in chat** — the whole thing, not a summary.
3. Save it as a dated file:

```bash
mkdir -p docs/open-source-audits
# write the report to docs/open-source-audits/YYYY-MM-DD-<slug>.md (default slug: full-audit)
```

**Never overwrite an existing dated report.** If a report for today's date and slug already exists, append a numeric suffix (`-2`, `-3`) rather than replacing it — each audit is its own dated record, so a maintainer can `git diff` across dates to see readiness improve or regress. Do **not** `git add`, commit, or push it — tell the user it's there and unstaged. If the project's `.gitignore` ignores `docs/`, say so instead of working around it.

## Re-auditing

When asked to re-check after readiness work was done, find the most recent file under `docs/open-source-audits/` (by filename date), run the full audit again (don't diff the old report instead of re-grading — a fix can regress another check), and add the `Progress since <date>` line from `references/output-format.md` → section 1, naming how many checks improved and how many regressed. Write a *new* dated file; never overwrite the previous one.
