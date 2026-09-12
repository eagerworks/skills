# Repo Handoff — Workflow

Run the phases in order. The only file you write is the report (Phase 5). The only commands you execute are the read-only probes and, optionally, the project's own lint/typecheck/test once (Phase 2).

## Phase 0 — Config

Read `.eagerworks/repo-handoff.json` if present (schema: `references/config.md`). Defaults: report at `docs/repo-handoff.md`, all ten dimensions enabled, command execution allowed. Disabled dimensions are listed in the report footer — never silently dropped.

If `docs/repo-handoff.md` (or `reportPath`) already exists, read it now and keep every question whose `Answer:` line is non-empty — you will carry those answers over in Phase 5.

## Phase 1 — Discovery

Read-only. Build four inventories before grading anything:

```bash
# Stack and layout
ls -la
cat package.json Gemfile pyproject.toml go.mod 2>/dev/null
cat pnpm-workspace.yaml turbo.json 2>/dev/null
find . -maxdepth 3 -name Dockerfile* -o -maxdepth 3 -name docker-compose*.yml | grep -v node_modules

# Written knowledge — read every one of these in full
find . -maxdepth 2 \( -iname 'README*' -o -iname 'CONTRIBUTING*' -o -iname 'CHANGELOG*' -o -name 'AGENTS.md' -o -name 'CLAUDE.md' \) | grep -v node_modules
find docs doc wiki adr .github -type f 2>/dev/null
cat .env.example .env.sample .env.template 2>/dev/null

# History
git --no-pager log --oneline -n 30
git --no-pager log -1 --format='%ci'
git --no-pager shortlog -sn --no-merges --since='12 months ago'
git --no-pager branch -a --sort=-committerdate | head -30
git --no-pager tag --sort=-creatordate | head -20
git --no-pager log --format=%H --name-only --since='12 months ago' | grep -v '^$' | grep -v '^[0-9a-f]\{40\}$' | sort | uniq -c | sort -rn | head -25   # churn hot spots

# External touchpoints — every match is a service to inventory
grep -rEho 'ENV\[["'\''][A-Z0-9_]+' --include='*.rb' . | sort -u
grep -rEho 'process\.env\.[A-Z0-9_]+' --include='*.ts' --include='*.js' --include='*.tsx' . | grep -v node_modules | sort -u
grep -rEho 'os\.(environ|getenv)\(["'\''][A-Z0-9_]+' --include='*.py' . | sort -u
grep -rEil 'stripe|sentry|twilio|sendgrid|mailgun|ses|s3|cloudfront|firebase|auth0|okta|segment|mixpanel|datadog|newrelic|pusher|algolia|openai|anthropic' --include='*.rb' --include='*.ts' --include='*.js' --include='*.py' --include='*.yml' --include='*.json' . | grep -v node_modules | head -50

# Scheduled work
grep -rEl 'cron|schedule|whenever|sidekiq-scheduler|clockwork|celery beat' --include='*.rb' --include='*.yml' --include='*.yaml' --include='*.py' --include='*.ts' --include='*.json' . | grep -v node_modules
grep -rl 'schedule:' .github/workflows 2>/dev/null

# TODO density
grep -rEn 'TODO|FIXME|HACK|XXX' --include='*.rb' --include='*.ts' --include='*.tsx' --include='*.js' --include='*.py' --include='*.go' . | grep -v node_modules | wc -l
```

Also run the read-only `gh` probes when `gh auth status` succeeds; each failure becomes ⚪ with a question:

```bash
gh auth status
gh pr list -L 20 --state open --json number,title,author,updatedAt
gh issue list -L 30 --state open --json number,title,labels
gh api "repos/{owner}/{repo}/collaborators" --jq '.[].login' 2>&1 | head
gh api "repos/{owner}/{repo}/actions/secrets" --jq '.secrets[].name' 2>&1 | head   # names only, never values
```

## Phase 2 — Verify (the only execution step)

You may **run** a command only if all of these hold:

- It is lint, typecheck, unit test, build, or a read-only dependency audit — never setup, install, migrate, seed, deploy, publish, release, or anything touching `docker`, `db:`, `prisma migrate`, `terraform apply`, `--force`, `rm`.
- You have read its definition and it doesn't shell out to any of the above.
- It has a non-interactive form (`CI=1`, `--ci`, `vitest run`, `--no-watch`, `-q`).
- Dependencies are already installed (`node_modules/`, `vendor/bundle`, `.venv`) — **do not install them**; if missing, grade 3.2 ⚪.
- Config has not set `runCommands: false`.

```bash
# ✅ correct
CI=1 timeout 900 bash -c 'time npx vitest run' 2>&1 | tail -20
CI=1 timeout 900 bash -c 'time bundle exec rspec --format progress' 2>&1 | tail -20
timeout 300 bash -c 'time npx tsc --noEmit' 2>&1 | tail -5
npm audit --omit=dev 2>&1 | tail -15          # read-only
bundle exec bundle-audit check 2>&1 | tail -15 # read-only, only if the gem is already installed
npm outdated 2>&1 | head -30                   # read-only

# ❌ wrong
bin/setup                    # may create DBs, install tools, prompt
npm install                  # mutates node_modules and possibly the lockfile
bundle exec rails db:migrate # mutates a database
docker compose up
terraform plan               # may need cloud credentials and can hit rate limits
npm audit fix                # edits the lockfile
```

Secret scanning is read-only and value-blind:

```bash
# ✅ correct — locations only
git --no-pager log -p --all -S 'BEGIN PRIVATE KEY' --format='%H %s' | grep -E '^[0-9a-f]{40}' | head
git --no-pager log --all --diff-filter=A --name-only --format='' | grep -Ei '(^|/)\.env$|\.pem$|\.p12$|credentials\.json$|service-account.*\.json$' | sort -u
gitleaks detect --no-git --report-format json --redact 2>/dev/null | head   # only if already installed; --redact is mandatory

# ❌ wrong
cat .env                    # never print secret values into the chat or the report
```

Record what ran, exit codes, and runtime; they go in the footer.

## Phase 3 — Grade

For each enabled dimension, walk every check in `references/dimensions.md`, assign a grade, and attach evidence (`file:line`, a command and its output, or the exact contradiction between docs and code). Apply the conservatism rule: doubt → the grade that produces a question.

## Phase 4 — Write the questions

For every check graded 🔴, 🟡, or ⚪:

1. Take the matching question(s) from the bank and specialize them with what you found (`For Stripe (found in config/initializers/stripe.rb:3) …`).
2. Assign a priority (P0/P1/P2) starting from the check's default; raise it when the evidence warrants.
3. State the evidence gap in one line under *Why we ask*.
4. Leave `Answer:` empty — unless Phase 0 recovered an answer for the same question text.

Merge duplicates: one service found in five files is one set of questions, not five. Sort by priority, then by dimension. Number sequentially across the whole report so the handoff meeting can reference "Q12".

## Phase 5 — Deliver: chat first, then the file

1. Fill `assets/handoff-report.md` (format: `references/output-format.md`).
2. **Print the complete report in chat** — the whole thing, not a summary.
3. Save the identical markdown:

```bash
mkdir -p docs
# write the report to docs/repo-handoff.md (or reportPath from config)
```

Overwrite an existing report after carrying over answers (Phase 0). Do **not** `git add`, commit, or push it — tell the user it's there and unstaged. If the project's `.gitignore` ignores `docs/`, say so instead of working around it.

## Re-running after the previous team answered

The intended loop: run → send the questions → the previous team fills in `Answer:` lines (in the file, in a meeting, or by email) → someone pastes the answers into the file → run again. On the second run, answered questions keep their answers, checks whose answer resolves the gap can be regraded (state that the grade comes from the answer, not the code), and only the still-open questions remain highlighted in the **Open questions** count in the header.
