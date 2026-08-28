# Loop Engineering Audit — Rubric

The authoritative checklist. Each dimension lists concrete checks, what evidence satisfies them, and the grade decision. Grade every check; roll up each dimension to its worst check.

Grades: 🔴 **Blocker** · 🟡 **Gap** · 🟢 **Ready** · ⚪ **Unverifiable from code**. Definitions and the verdict rule are at the end.

## Dimension 1 — Agent-facing context

*Can an agent learn how this project works without asking a human?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 1.1 Agent instructions file exists | `AGENTS.md` or `CLAUDE.md` (or `.cursor/rules/`, `.github/copilot-instructions.md`) at the root | 🔴 none of them exist |
| 1.2 States the stack and layout | Names the framework/versions and the directories that matter | 🟡 generic or missing |
| 1.3 States the verification commands | The exact lint/typecheck/test/build commands, with the non-interactive form | 🔴 missing; 🟡 present but one is wrong or interactive |
| 1.4 States conventions | Naming, file placement, testing style, commit format — the things a reviewer would otherwise correct | 🟡 missing |
| 1.5 States forbidden / dangerous actions | What never to run or edit (migrations by hand, prod env, generated dirs) | 🟡 missing |
| 1.6 Is accurate | Every command and path in it exists on disk | 🔴 a documented command doesn't exist or fails |
| 1.7 README setup matches reality | Steps in `README.md`/`CONTRIBUTING.md` reference files and scripts that exist | 🟡 stale |

Full checklist and a starter: `references/agent-docs.md`, `assets/AGENTS.example.md`.

```bash
# ✅ correct — verify each documented command exists before crediting it
grep -oE '`(bin/[a-z-]+|npm run [a-z:-]+|pnpm [a-z:-]+|bundle exec [a-z]+|make [a-z-]+)`' AGENTS.md | sort -u
# then: ls bin/, jq .scripts package.json, make -n <target>

# ❌ wrong — grading 1.6 🟢 because the file "looks thorough"
```

## Dimension 2 — Reproducible environment

*Can an agent get from clone to passing tests without a human?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 2.1 Toolchain pinned | `.tool-versions` / `.ruby-version` / `.node-version` / `.nvmrc` / `engines` / `.python-version` / `go.mod` go directive | 🟡 unpinned |
| 2.2 Lockfile committed | `Gemfile.lock`, `package-lock.json`/`pnpm-lock.yaml`/`yarn.lock`, `poetry.lock`/`uv.lock`, `go.sum` — and not in `.gitignore` | 🔴 missing or ignored |
| 2.3 One-command setup | `bin/setup`, `make setup`, `npm run setup`, devcontainer, or a documented ordered script | 🔴 setup is a prose list of manual steps; 🟡 script exists but is undocumented |
| 2.4 Setup is non-interactive | No prompts; env comes from `.env.example` / defaults | 🔴 prompts for input |
| 2.5 Environment template | `.env.example` (or equivalent) listing every variable, with safe local defaults | 🟡 missing or incomplete vs. `ENV[...]`/`process.env.*` usage |
| 2.6 Services declared | DB/Redis/etc. via `docker-compose*.yml`, devcontainer, or documented local install | 🟡 undocumented; 🔴 tests need a service nobody declares |
| 2.7 No secrets required for local test run | Tests pass with `.env.example` values | 🔴 tests need a real API key |

Rails example: `bin/setup` that calls `bin/rails db:prepare` and reads `config/database.yml` with `ENV.fetch("DATABASE_URL") { "postgres://localhost/app_development" }` → 🟢. Node example: `"postinstall": "husky"` + `.nvmrc` + `pnpm-lock.yaml` → 2.1–2.2 🟢; missing `.env.example` while `src/` reads 14 `process.env.*` keys → 2.5 🟡.

## Dimension 3 — Fast deterministic verification

*Can an agent tell, by itself, whether its change is correct?*

Grade each of **lint**, **typecheck** (where the language has one), **unit tests**, **build** separately.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 3.1 Command exists | A script/task for it | 🔴 no test command at all; 🟡 no lint or no typecheck |
| 3.2 Documented | Named in `AGENTS.md`/`CLAUDE.md`/README | 🟡 exists but undocumented |
| 3.3 Non-interactive | Runs to completion with no TTY, no watch, no prompt | 🔴 default form is watch/interactive with no documented alternative |
| 3.4 Exit code | Non-zero on failure (not `|| true`, not `"echo skipped"`) | 🔴 always exits 0 |
| 3.5 Wall time | Unit suite ≤ 5 min, lint/typecheck ≤ 2 min (default budgets; override via config) | 🟡 over budget; 🔴 > 20 min with no focused subset documented |
| 3.6 Deterministic | No `skip`/`pending`/`xit`/`.only` accumulation, no `retry` wrappers hiding flakes, no wall-clock/network dependence in unit tests | 🟡 evidence of flakiness or skipped tests |
| 3.7 Focused runs possible | A single file/example can be run (`rspec path:line`, `vitest run path`, `pytest path::test`) and it's documented | 🟡 undocumented |

```bash
# ✅ correct — non-interactive forms an agent can loop on
CI=1 npx jest --ci
npx vitest run
bundle exec rspec --format progress
pytest -q
# ❌ wrong — these hang an unattended loop
npx jest --watch
npx vitest          # watch mode by default outside CI
bin/rails test -p   # pager/interactive on some setups
```

`"test": "echo \"Error: no test specified\" && exit 1"` is 🔴 (3.1), not 🟢 on 3.4.

## Dimension 4 — Test coverage as a safety net

*Does a green suite actually protect against a wrong change?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 4.1 Tests exist | A test tree with more than fixtures/smoke | 🔴 zero tests |
| 4.2 Cover the change surface | The dirs an agent will edit (controllers/services/models, `src/` modules, components, API routes) have sibling tests | 🟡 large areas untested |
| 4.3 Assertions are real | Tests assert behaviour, not just "doesn't raise" / snapshot-everything | 🟡 assertion-free or snapshot-only |
| 4.4 Integration/e2e presence | Request/system/API-level tests exist for the main flows | 🟡 unit-only |
| 4.5 Coverage tooling | `simplecov`, `c8`/`istanbul`, `coverage.py` configured, ideally with a threshold | 🟡 none (informational, never 🔴) |

Evidence: ratio of source dirs to test dirs (`ls app/services | wc -l` vs `ls spec/services | wc -l`), `grep -rL "expect\|assert" spec/ test/` for assertion-free files, `grep -rn "\.only\|xit(\|skip(\|pending" test/`.

## Dimension 5 — Task definition surface

*Can an agent be handed a unit of work it can finish and prove finished?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 5.1 Issue template with acceptance criteria | `.github/ISSUE_TEMPLATE/*` (or docs) with an AC / "definition of done" section | 🟡 missing |
| 5.2 PR template | `.github/PULL_REQUEST_TEMPLATE.md` with checklist (tests, docs) | 🟡 missing |
| 5.3 Commit convention | Conventional Commits or equivalent, stated and visible in `git log` | 🟡 unstated |
| 5.4 Branch convention | Stated naming and base branch | 🟡 unstated; note the default branch from `git symbolic-ref refs/remotes/origin/HEAD` |
| 5.5 Board / labels | A project board or label taxonomy the loop can read status from (`gh project`, `gh label list`) | ⚪ if `gh` can't read; 🟡 none |
| 5.6 In practice | Sample recent issues/PRs (`gh issue list -L 10`, `gh pr list -L 10 --state merged`): do they carry AC / follow the template? | ⚪ if unreadable; 🟡 templates unused |

## Dimension 6 — CI & merge gates

*Does the loop have a finish line?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 6.1 CI exists | `.github/workflows/*.yml` (or `.gitlab-ci.yml`, `.circleci/`) running on PRs | 🔴 none |
| 6.2 CI runs the same checks as local | The workflow invokes the same lint/test/build commands the docs name — not a divergent set | 🟡 divergent (agent passes locally, fails in CI) |
| 6.3 CI is green on the default branch | `gh run list -b main -L 5` | ⚪ if unreadable; 🔴 consistently red |
| 6.4 Required status checks / branch protection | `gh api repos/{owner}/{repo}/branches/<default>/protection` | ⚪ if 403/404 with the question to ask; 🟡 none |
| 6.5 CI needs no secrets for the verify job | Lint/test jobs run on fork PRs without `secrets.*` | 🟡 verify job needs secrets |
| 6.6 Deploy / preview is observable | If deploys exist, their status is readable (`gh run`, a status check) | ⚪ / 🟡 |

## Dimension 7 — Guardrails & safety

*What happens when the agent gets it wrong?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 7.1 No secrets in the repo | `.gitignore` covers `.env*`, `config/master.key`, `*.pem`; `git log -p` sampling and `grep -rEn "(api|secret|token)[_-]?key.*['\"][A-Za-z0-9]{20,}"` are clean | 🔴 committed credential |
| 7.2 Destructive commands fenced | `db:drop`/`db:reset`, `prisma migrate reset`, deploy scripts guarded (env check, confirmation, or listed as forbidden in agent docs) | 🟡 unfenced and undocumented |
| 7.3 Agent tool permissions | `.claude/settings.json` `permissions.allow/deny`, or equivalent, listing safe commands and denying deploy/destroy | 🟡 absent (informational) |
| 7.4 Hooks | `pre-commit`/`husky`/`lefthook`/`overcommit` running lint or tests | 🟡 absent |
| 7.5 Generated / vendored dirs marked | `node_modules`, `vendor/`, `dist/`, generated clients listed in docs or `.gitattributes linguist-generated` | 🟡 unmarked and large |
| 7.6 Migrations / schema policy | Stated rule for how schema changes are made and reviewed | 🟡 unstated (Rails/Prisma/Django only) |

## Grade ladder

- 🔴 **Blocker** — an unattended loop would hang, guess, or be unable to verify. Any 🔴 ⇒ verdict **Not ready**.
- 🟡 **Gap** — the loop runs but burns human attention or agent iterations. Only 🟡 ⇒ **Partially ready**.
- 🟢 **Ready** — a concrete rule was checked and satisfied.
- ⚪ **Unverifiable from code** — always carries the exact question or setting a human must confirm. ⚪ never changes the verdict but always appears in the report.

## Conservatism rule

A grade other than 🟢 must cite evidence: a path, a line, a command with its output, or a documented claim that turned out false. If you can't produce that, the check is either 🟢 (you checked and it's fine) or ⚪ (you couldn't check) — never 🟡 on a hunch. Never 🟢 something you didn't check.

Config `.eagerworks/loop-engineering-audit.json` may disable a dimension or change the time budgets in 3.5 — any disabled dimension is disclosed in the report, never silently omitted (`references/config.md`).
