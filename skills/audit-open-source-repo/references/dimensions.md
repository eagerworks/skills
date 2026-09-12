# Audit Open Source Repo — Dimensions, Checks, and Grades

The authoritative checklist. Each dimension lists concrete checks, what evidence satisfies them, and the grade decision. Grade every check; roll up each dimension to its worst check. `SKILL.md` only summarizes this file — it is the single source of truth.

Grades: 🔴 **Barrier** · 🟡 **Friction** · 🟢 **Clear** · ⚪ **Unverifiable from code**. The grade ladder, the exhaustive 🔴 list, the two capped dimensions, and the conservatism rule are at the end.

Dimension order **is the newcomer's journey** — 1 through 10 trace Land → Decide it's alive → Read the contract → Set up → Pick something → Orient → Meet the standards → Prove it → Get the gate green → See it land. `references/first-contribution.md` maps its eight stages onto these check ids verbatim, and the Contributor Readiness Plan's ordering rule (`references/output-format.md`) sorts P0 rows by this same order.

## Dimension 1 — Discoverability & first impression

*Can a stranger tell, in the first minute, what this is, whether it is alive, and whether they may legally change it?*

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 1.1 README states what and for whom | `README.md` at root; its first 30 lines name the problem the project solves and who it's for — not only the framework and the badges | 🔴 no README, or a stub under 10 lines; 🟡 present but never says what the project *is* (opens straight on install/badges) | P2 |
| 1.2 Runnable quickstart | An install/run block on the first screen, and every command in it resolves (a script in `package.json#scripts`, a `Makefile` target, a file in `bin/`) | 🔴 no usage or run instructions at all; 🟡 a named command doesn't exist — cite it | P2 |
| 1.3 LICENSE present and OSI-approved | `LICENSE`/`LICENSE.md`/`COPYING` at root with recognizable OSI text (MIT, Apache-2.0, BSD, GPL, MPL) and the placeholders filled in | 🔴 no license file — the default is all-rights-reserved and nobody may legally contribute; 🟡 non-OSI/source-available, or `[year] [fullname]` left unfilled. **n/a when `project.audience: "internal"`** | P0 |
| 1.4 License consistent across metadata | The `license` field in `package.json`/`pyproject.toml`/`Cargo.toml`/gemspec/`composer.json` and any README badge match the LICENSE file | 🟡 mismatch (cite both) — the contributor can't tell which terms they're agreeing to | P2 |
| 1.5 Status & maturity stated and true | README states alpha/beta/stable/maintenance, and the claim agrees with `git log -1 --format=%cs` on the default branch and the latest release date | 🟡 unstated; 🟡 "actively maintained" with the last default-branch commit over 12 months old — cite both dates | P2 |
| 1.6 Supported versions & platforms stated | Runtime/language versions named in README or metadata (`engines`, `python_requires`, `rust-version`, `required_ruby_version`) and they agree with the CI matrix | 🟡 unstated; 🟡 metadata and CI matrix disagree (cite both) | P2 |
| 1.7 Visual or demo evidence | A screenshot, GIF, asciinema recording, or hosted demo for a UI/CLI project; runnable examples for a library | 🟡 a user-facing project with nothing to look at. n/a for a headless library with code examples | P2 |
| 1.8 Repo description & topics | `gh repo view --json description,topics` returns a non-empty description and at least one topic | ⚪ if `gh` can't read the repo; 🟡 empty description or no topics. n/a when `project.audience: "internal"` | P2 |

## Dimension 2 — The contribution contract

*Does the repo state, in writing, how an outsider proposes work — and what will actually be accepted?*

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 2.1 CONTRIBUTING exists where the platform looks | `CONTRIBUTING.md` at root, `.github/`, or `docs/` (the three paths GitHub/GitLab surface it from), and linked from README | 🔴 absent from all three; 🟡 present in a path the platform won't surface (`meta/`, `wiki/`), or not linked from README | P0 |
| 2.2 The flow works for someone without push access | The document names fork → branch → PR (or, under `audience: "internal"`, branch → PR), with the upstream-sync step | 🔴 it assumes push access to the upstream repo — an external contributor cannot follow it; 🟡 the fork step is implied but never written | P0 |
| 2.3 Branch & commit conventions stated and observed | The stated rules match the repo's own history — `git log --oneline -30`, `git branch -r` | 🟡 unstated; 🟡 stated and the last 30 commits don't follow them — cite the ratio | P1 |
| 2.4 What gets accepted is stated | A scope statement: kinds of change welcomed, kinds that need discussion first (new dependency, new public API, breaking change, large refactor), what's out of scope | 🟡 missing — paid in whole wasted feature PRs; 🟡 stated but contradicted by recently closed-unmerged PRs (⚪ if `gh` can't read) | P1 |
| 2.5 Review expectations stated | Who reviews, how many approvals are needed, a response-time expectation — even "we aim for a week; ping after two" | 🟡 unstated | P2 |
| 2.6 Contribution-agreement stance explicit and consistent | Either (a) a DCO/CLA is documented in CONTRIBUTING **and** enforced by a check (`.github/workflows/dco*.yml`, a CLA-assistant status check, `Signed-off-by:` throughout history), or (b) nothing in the repo requires or implies one | 🔴 a CLA/DCO check runs on PRs and no document mentions it — the PR is blocked by a rule the contributor never saw; 🟡 documented but unenforced, or scattered `Signed-off-by:` with no stated policy | P0 / P1 |
| 2.7 Where to ask before writing code | A named channel (Discussions, an issue type, Discord/Slack/Matrix, a mailing list) with a resolvable link, in README or CONTRIBUTING | 🟡 missing; 🟡 an invite-only or dead link — note it, don't credit it | P2 |
| 2.8 The contract matches the code | Every backticked command, path, script, file and env var in CONTRIBUTING (and README's setup section) resolves against the tree | 🔴 a named setup or verification command does not exist; 🟡 a named path or doc does not exist | P0 |

## Dimension 3 — Community health & governance

*Does a contributor know who decides, where to ask, and what happens when something goes wrong?*

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 3.1 Code of conduct with a real reporting contact | `CODE_OF_CONDUCT.md` in root/`.github`/`docs`, with an address or form a person can actually use | 🟡 absent; 🟡 present with a placeholder (e.g. the Contributor Covenant's `[INSERT CONTACT METHOD]`) still in it — a policy nobody can invoke. n/a when `project.audience: "internal"` | P2 |
| 3.2 Private vulnerability reporting route | `SECURITY.md` naming a private route (a security email, or GitHub private vulnerability reporting) and which versions get fixes | 🔴 the documented route is "open an issue" — the repo instructs public disclosure; 🟡 absent. n/a (route becomes an internal channel) when `project.audience: "internal"` | P0 / P2 |
| 3.3 Maintainers are identifiable | `CODEOWNERS`, `MAINTAINERS.md`, or a README section naming at least one human or team | 🟡 nobody named — review assignment falls to chance and there is no one to ping | P2 |
| 3.4 CODEOWNERS is valid and covers the feature surface | Every pattern matches at least one tracked path (`git ls-files` per pattern), and the directories features land in are covered | 🟡 patterns matching nothing (stale), or the main source dirs uncovered; ⚪ owner-handle existence without `gh` | P2 |
| 3.5 Decision-making & merge authority stated | `GOVERNANCE.md` or a CONTRIBUTING section: who can merge, how disagreements resolve, how someone becomes a maintainer | 🟡 unstated where `git shortlog -sn --no-merges` over 12 months shows more than one significant contributor. 🟢 for a single-maintainer project that says so in one sentence | P2 |
| 3.6 Questions are routed away from the bug tracker | `SUPPORT.md`, or `.github/ISSUE_TEMPLATE/config.yml`'s `contact_links` pointing questions elsewhere | 🟡 missing — the tracker fills with support requests and entry-point issues get buried | P2 |
| 3.7 Health files are where the platform looks | CoC, SECURITY, SUPPORT, templates resolve from root, `.github/`, or `docs/` | 🟡 a health file in a non-standard path where neither the platform nor a contributor will find it | P2 |

## Dimension 4 — Time to first build

*Can a contributor who has none of the team's secrets get from `git clone` to a running app and a green test run?*

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 4.1 One-command bootstrap | `bin/setup`, `script/bootstrap`, `make setup`, `npm run setup`, a devcontainer, or a documented ordered block — the entrypoint exists and is executable | 🔴 setup is prose with more than five manual steps; 🟡 a script exists but is undocumented | P0 |
| 4.2 Prerequisites enumerated and pinned | Runtimes pinned (`.tool-versions`, `.nvmrc`, `.ruby-version`, `engines`, `rust-toolchain.toml`, Dockerfile base) and system packages named (libpq, ImageMagick, protoc, build tools) | 🔴 nothing pinned and no documented versions; 🟡 partially pinned, or system deps undocumented | P0 |
| 4.3 Lockfile committed and not ignored | The lockfile is tracked and `.gitignore` doesn't exclude it | 🔴 missing or ignored — the contributor builds against a different dependency graph than CI | P0 |
| 4.4 `.env.example` covers what the code reads | Keys in `.env.example` (or equivalent template) are a superset of the variables the app and the suite read (`process.env.X`, `ENV[...]`, `os.environ[...]`) | 🔴 the app or the suite can't boot without a variable nobody documents; 🟡 keys missing from the template — list them | P0 |
| 4.5 No contributor-unobtainable secret on the default path | Running the app and the suite locally needs no credential only an employee can get | 🔴 the documented path needs a private registry (`.npmrc`/`.yarnrc.yml` scoped registry, a `Gemfile` private source, `pip.conf` index-url, a `git+ssh` dep), a VPN, cloud SSO, `docker login`, or a prod data snapshot; 🟡 optional features do and it's documented as optional | P0 |
| 4.6 Local services declared and self-starting | `docker-compose.yml`/`compose.yaml`, a devcontainer, or documented installs for every DB/queue/cache/search the code connects to | 🔴 the suite needs a service nobody declares; 🟡 declared but undocumented | P0 |
| 4.7 Seeds or fixtures give a usable local app | `db/seeds.*`, `prisma/seed.ts`, a fixtures tree, a `seed` script, or documented sample data | 🟡 missing in an app with a UI or a data model — the contributor can't see whether their feature works | P1 |
| 4.8 Setup is non-interactive and platform-honest | No prompts on the happy path; if only one OS is supported, it says so | 🟡 prompts for input; 🟡 macOS-only (`brew`) instructions with no Linux/WSL note | P1 |

## Dimension 5 — Task surface *(capped — never 🔴)*

*Is there something specific, scoped, and unclaimed that a newcomer could pick up today?*

No check here is ever 🔴 — see "Capped dimensions" below.

| Check | 🟢 when | 🟡 / ⚪ when | P |
|---|---|---|---|
| 5.1 Issue templates exist | `.github/ISSUE_TEMPLATE/*.yml` (forms) or `*.md` with front-matter, covering at least bug and feature/enhancement | 🟡 none. Forms are the stronger artifact; markdown-only isn't a gap by itself | P2 |
| 5.2 Templates ask for what a maintainer needs | Bug: repro, version, environment. Feature: problem, proposed solution, alternatives considered | 🟡 a template that is one free-text box | P2 |
| 5.3 Templates are actually used | Sample `gh issue list -L <history.issueSample> --json body,createdAt`: recent bodies carry the template's headings | 🟡 fewer than half of recent issues follow them — cite the ratio; ⚪ if `gh` can't read | P2 |
| 5.4 PR template with a contributor checklist | `.github/PULL_REQUEST_TEMPLATE.md` (or the directory form) asking for the linked issue, what changed, how it was tested, and docs | 🟡 absent; 🟡 present and unused across the last 10 merged PRs; ⚪ unreadable | P1 |
| 5.5 Entry-point labels exist and are populated | `gh label list` has `good first issue`/`help wanted` (or equivalents) and `gh issue list --label "good first issue" --state open` returns at least one | 🟡 the label exists with zero open issues — a promise the repo doesn't keep; 🟡 no such label; ⚪ unreadable | P2 |
| 5.6 Entry-point issues are actionable | Read up to 5: each states expected behaviour, points at a file/area or gives a reproduction, and is unassigned with no stale claim | 🟡 one-liners, already assigned, or "I'll take this" older than 60 days with no PR — quote the issue numbers you read | P2 |
| 5.7 Direction is visible | A roadmap doc, a milestone with open issues, or a pinned "what we're working on" issue | 🟡 none — a feature contributor can't tell whether their idea is wanted before building it | P2 |
| 5.8 Issue triage is alive | Of the last `history.issueSample` issues: the share with any maintainer response, and the age of the oldest unanswered one — reported as numbers | 🟡 a majority unanswered, or a median first response beyond CONTRIBUTING's own stated expectation; ⚪ unreadable. *(PR-side responsiveness lives in 10.2–10.3; never duplicate that here.)* | P2 |

## Dimension 6 — Code orientation for feature work

*Can a contributor find where a new feature of the usual kind goes, and build it the way the team would have?*

No check here is 🔴 for absence — a missing map costs quality, it doesn't shut the door. The two 🔴s fire only on **contradiction** (a documented structure that doesn't exist) and on **unmarked generated code**, both of which send a contributor's PR down a path that will be rejected.

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 6.1 Architecture / module map | A docs page, README section, or diagram naming the top-level components and what lives where — and every path it names exists | 🔴 the map names directories or modules that don't exist (cite two); 🟡 no map at all for a repo with more than a handful of source dirs | P1 |
| 6.2 A worked path for the common feature kind | The docs name, for at least one common change (new API endpoint, new screen, new CLI command, new provider/adapter), the files to touch and in what order | 🟡 missing — every contributor reverse-engineers it from `grep`, and each guesses differently | P1 |
| 6.3 Extension points are named | Plugin/adapter/hook registries, DI containers, `register*` functions, "add yours here" tables — with the file that holds the registry | 🟡 an extension system exists in code (a registry map/array is the evidence) and is documented nowhere | P1 |
| 6.4 Layout & naming conventions stated and observed | File placement, module/class naming, feature-folder vs. layer — written down and matching the tree | 🟡 unstated; 🟡 stated and the tree disagrees — cite two counterexamples | P1 |
| 6.5 Decisions are recorded | ADRs, design docs, or dated decision notes for non-obvious choices (custom auth, state management, event bus, a hand-rolled framework) | 🟡 none, and the repo contains machinery a newcomer would otherwise "fix" | P2 |
| 6.6 Generated & vendored trees are marked | Generated dirs identified (`.gitattributes linguist-generated`, a header comment, or a documented `generate` command) | 🔴 generated files are committed with no marker and no regenerate command — the contributor hand-edits them and the review bounces; 🟡 marked but the regenerate command is undocumented | P0 / P1 |
| 6.7 Schema & migration convention stated | How to add a migration, whether the checked-in schema file is edited by hand, reversibility/squash policy | 🟡 unstated in a project with a migrations directory | P1 |
| 6.8 Public surface & compatibility rules stated | What counts as public (exported modules, CLI flags, response shapes, DB columns), what a contributor may not break, and the deprecation policy | 🟡 unstated for a library, SDK, or public API | P1 |

## Dimension 7 — Guidelines as an executable spec

*Are the project's standards enforced by a command the contributor can run, instead of by a reviewer's memory?*

This is the heart of the audit. A mandatory sub-table accompanies this dimension's findings — see `references/output-format.md` → "The enforcement matrix".

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 7.1 Formatter configured, committed, check-only command exists | A formatter config in the repo (`.prettierrc`, `rustfmt.toml`, `black`/`ruff format` in `pyproject.toml`, `.rubocop.yml`, gofmt implied), plus `.editorconfig`, plus a documented command that *checks without writing* (`--check`, `--dry-run`, `format:check`) | 🟡 no formatter; 🟡 config present but no check-only command; 🟡 formatting rules that exist only in `.vscode/settings.json` with no CLI equivalent — a fork clone never gets them | P1 |
| 7.2 Linter configured and runnable over the whole repo | A lint config committed (`eslint.config.*`, `.rubocop.yml`, `ruff`/`flake8`/`pylint`, `.golangci.yml`, clippy config) and one documented non-interactive command covering the whole tree | 🔴 no linter and no formatter of any kind for the primary language — style is decided in review, exactly the cost this audit exists to remove; 🟡 present but undocumented, or scoped to one subdirectory while the rest is unchecked | P0 / P1 |
| 7.3 Typecheck where the language has one | `tsconfig.json` with `strict` (or a stated, dated exception), `mypy`/`pyright` config, Sorbet/RBS — with a documented command | 🟡 the language has a checker and the project runs none; 🟡 `strict: false` with no note saying why | P1 |
| 7.4 Standards run before the push, automatically | `.husky/`, `lefthook.yml`, `.pre-commit-config.yaml`, `.overcommit.yml`, or a `core.hooksPath` hook — and installing it is part of bootstrap (`prepare`/`postinstall`/`bin/setup`) | 🟡 no hook; 🟡 hook config present but nothing installs it during setup — a fresh fork clone silently has no hooks | P1 |
| 7.5 One command runs every standard | A single aggregate entry point (`make check`, `npm run check`, `bin/ci`, `just check`) running format-check + lint + typecheck + tests, named in CONTRIBUTING | 🟡 missing — the contributor must remember four commands and will forget one. Never 🔴 | P1 |
| 7.6 Written rules and enforced rules agree | Every prose rule in the style guide is either in the linter config or explicitly marked "not enforced — please follow" | 🟡 prose rules the linter doesn't enforce but a reviewer will (list two); 🟡 a linter rule that contradicts the prose | P1 |
| 7.7 The standards pass on an untouched default branch | Run the documented check-only commands once on a clean checkout: each exits 0 | 🔴 lint or format-check fails on code nobody touched — a contributor cannot tell their own failures from the project's, and the first thing they learn is that the checks are noise; ⚪ when `runCommands: false` or dependencies aren't installed | P0 |
| 7.8 Commit-message convention is enforced, not merely requested | If CONTRIBUTING asks for Conventional Commits or a sign-off, a commitlint config, hook, or CI job enforces it | 🟡 asked for and unenforced. 🟢 both when enforced and when nothing is asked for | P1 |

## Dimension 8 — Self-verification before the push

*Can a contributor prove their feature is done and correct before a maintainer looks at it?*

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 8.1 Test command exists, documented, non-interactive | A real script/target, named in CONTRIBUTING/README, running to completion with no TTY, no watch, no prompt | 🔴 none, or `"test"` is `echo`/`exit 0`/`\|\| true`; 🟡 exists but the documented form is the watch form | P0 |
| 8.2 The suite runs without credentials or the network | Passes on `.env.example` values with only the declared local services | 🔴 the suite needs a real API key or a live external service no compose file provides; 🟡 a subset does and is neither marked nor skippable | P0 |
| 8.3 A focused run is documented | How to run one file or one test (`vitest run <path>`, `pytest path::test`, `rspec path:line`, `go test ./pkg/...`) | 🟡 undocumented — the contributor runs the full suite for a one-line change, then stops running it at all | P1 |
| 8.4 Suite wall time known and bounded | Measured once, at or under `budgets.testSeconds` (default 600) | 🟡 over budget with no documented fast subset; ⚪ when not run. Never 🔴 — slowness is friction here, not a closed door | P1 |
| 8.5 Test conventions are written and copyable | Docs say where tests live, how to name them, which helpers/factories/fixtures to use — and point at an exemplar test file that exists | 🟡 missing — the contributor invents a style the reviewer then rejects, the single most common cause of a good feature PR going stale | P1 |
| 8.6 The expected test surface for a feature is stated | The definition of done: which layers a new feature must cover (unit + request/integration? a story? a migration test?) | 🟡 unstated — "needs tests" in review is unactionable without it | P1 |
| 8.7 Coverage expectation stated and measurable | Coverage tooling configured with either an enforced threshold or an explicit "no hard threshold" statement | 🟡 a threshold in prose that nothing measures; 🟡 no coverage tooling (informational, never 🔴) | P1 |
| 8.8 Visual/manual evidence expectation for UI work | For a project with a UI: the docs and the PR template say what to attach (before/after screenshot, an accessibility note, manual steps) | 🟡 a UI project with no stated visual-evidence expectation. n/a for headless projects | P1 |

## Dimension 9 — Fork-safe CI & merge gates

*Does the automated gate actually run — and pass — on a pull request from a fork?*

A mandatory sub-table accompanies this dimension's findings — see `references/output-format.md` → "The fork-gate table".

| Check | 🟢 when | 🔴 / 🟡 when | P |
|---|---|---|---|
| 9.1 CI triggers on `pull_request` | `.github/workflows/*.yml` (or `.gitlab-ci.yml`, `.circleci/config.yml`) with `on: pull_request` covering the default branch — not `push`-only | 🔴 no CI on PRs: nothing verifies an outsider's branch, and the whole review load lands on a human | P0 |
| 9.2 The basic gate needs no secrets | The lint/typecheck/test jobs reference no `secrets.*` beyond `GITHUB_TOKEN` and no protected `environment:` | 🔴 the verification job reads a secret — on a fork PR secrets are empty, so the gate fails for a reason the contributor can't fix or diagnose; 🟡 an optional job (coverage upload, preview deploy) fails on forks without `continue-on-error` or a fork guard | P0 |
| 9.3 Fork code is not run with write-scoped secrets | No workflow combines `pull_request_target` with a checkout of `head.sha`/`head.ref` | 🔴 present — it both hands an outsider the repo's secrets and proves the fork path was never designed | P0 |
| 9.4 CI runs the same checks the contributor runs | The workflow's `run:` steps invoke the commands CONTRIBUTING names (ideally the one aggregate command from 7.5) | 🟡 divergent — green locally, red in CI, and the contributor can't reproduce the failure. Cite both lists | P1 |
| 9.5 First-time-contributor run approval is accounted for | The repo states that a maintainer must approve the first workflow run from a new contributor, or the setting is known | ⚪ — not visible from code. Question: "Is *Require approval for first-time contributors* enabled, and who clicks it?" — never grade the fork path 🟢 while ignoring this | P2 |
| 9.6 The default branch is green | `gh run list -b <default> -L 10` shows recent runs passing | 🔴 persistently red — a contributor can't distinguish their break from the project's; ⚪ unreadable | P0 |
| 9.7 Merge is actually gated | `gh api repos/{owner}/{repo}/branches/<default>/protection` shows required status checks | ⚪ on 403/404, with the question; 🟡 no required checks — a green CI that doesn't gate merge teaches everyone the checks are decorative | P1 |
| 9.8 CI is fast enough and its failures are readable | Median duration of the last 10 runs at or under `budgets.ciMinutes` (default 20), and a failing job's log names the failing test or rule rather than a bare non-zero exit | 🟡 over budget, or a failure log names no file/rule; ⚪ unreadable | P1 |

## Dimension 10 — Review & release loop *(capped — never 🔴, graded from history only)*

*If a contributor opens a good PR, does anything happen to it — and does their merged work reach users?*

No check here is ever 🔴 — see "Capped dimensions" below. **Grading rule for this whole dimension: evidence is `git`/`gh` history, never a document.** A sentence in CONTRIBUTING is the *expectation* (it sets the threshold for 10.2); the history is the grade. Findings are reported as numbers, never as verdicts on people. A mandatory sub-table accompanies this dimension's findings — see `references/output-format.md` → "The stewardship metrics table".

| Check | 🟢 when | 🟡 / ⚪ when | P |
|---|---|---|---|
| 10.1 External contributions get merged | Of the last `history.prSample` merged PRs, at least one is from an author outside the maintainer set (`history.maintainers`, else CODEOWNERS union the top committers) | 🟡 zero external PRs merged in 12 months **while external PRs were opened** — cite both counts; 🟢 with a note when no external PR was ever opened; ⚪ unreadable | P2 |
| 10.2 Time to first review is measured | Median from `createdAt` to the first review or maintainer comment across the last `history.prSample` PRs — reported as a number whatever the grade | 🟡 beyond the expectation CONTRIBUTING states, or beyond `budgets.firstReviewDays` (default 14) when none is stated; ⚪ unreadable | P2 |
| 10.3 The open-PR queue isn't a graveyard | No open PR older than `budgets.staleOpenPrDays` (default 90) sits without a maintainer response, and at most half of open PRs are unanswered | 🟡 otherwise — cite the count and the oldest age; ⚪ unreadable | P2 |
| 10.4 A review rubric exists | What reviewers check (correctness, tests, docs, performance, migration safety) is written down, so a contributor can self-review first | 🟡 unwritten — the bar exists only in reviewers' heads, exactly what makes a first feature PR take five rounds | P1 |
| 10.5 Merge strategy & base branch stated and consistent | Squash/merge/rebase policy stated and matching `git log --merges` on the default branch; the base branch for PRs is named | 🟡 unstated; 🟡 stated and history disagrees | P2 |
| 10.6 Versioning policy stated | SemVer (or a stated alternative) declared, plus what a contributor must do for a breaking change (a changeset, a label, a migration note) | 🟡 unstated for a published artifact | P2 |
| 10.7 Changelog practice is real | `CHANGELOG.md` whose latest entry corresponds to the latest tag/release, or generator config (`.changeset/`, `release-please`, `release-drafter`), and the contributor's obligation is stated | 🟡 a changelog several releases behind — cite the last entry and the latest tag; 🟡 absent for a published artifact | P2 |
| 10.8 Releases actually happen | `gh release list`/`git tag --sort=-creatordate`: a release in the last 12 months, plus whether merge → release is automated | 🟡 no release in over 12 months for a project claiming active status (pair with 1.5); ⚪ unreadable | P2 |

---

## Grade ladder

- 🔴 **Barrier** — a first-time outside contributor is stopped, or sent down a path that cannot succeed. Any 🔴 ⇒ verdict **Closed to contributors**.
- 🟡 **Friction** — they get through, but it costs them or the maintainer time the repo could have saved, or it lets a feature land below the project's own bar. Any 🟡 with no 🔴 ⇒ verdict **Open with friction**.
- 🟢 **Clear** — a concrete rule was checked and satisfied.
- ⚪ **Unverifiable from code** — needs a repo setting, a dashboard, a private channel, or a person. Never moves the verdict; always produces a question.

## The exhaustive 🔴 list

Every condition that may ever be graded 🔴, and nothing else:

| Check | Fires when |
|---|---|
| 1.3 | No license file (public audience only) |
| 2.1 | No `CONTRIBUTING.md` anywhere the platform looks |
| 2.2 | The documented flow assumes upstream push access |
| 2.6 | A CLA/DCO check runs on PRs and no document mentions it |
| 2.8 | A setup or verification command named in the docs does not exist |
| 3.2 | The documented vulnerability route is a public issue (public audience only) |
| 4.1 | Setup is prose-only with more than five manual steps |
| 4.2 | No pinned or documented toolchain versions |
| 4.3 | Lockfile missing or gitignored |
| 4.4 | A variable the app/suite requires is documented nowhere |
| 4.5 | The default path needs a credential an outsider can't obtain |
| 4.6 | The suite needs an undeclared service |
| 6.1 | The architecture map names paths that don't exist |
| 6.6 | Committed generated code with no marker and no regenerate command |
| 7.2 | No linter and no formatter of any kind for the primary language |
| 7.7 | Lint or format-check fails on an untouched default branch |
| 8.1 | No test command, or a fake one |
| 8.2 | The suite needs real credentials or a live external service |
| 9.1 | No CI on `pull_request` |
| 9.2 | The basic verification job needs secrets |
| 9.3 | `pull_request_target` checking out fork code |
| 9.6 | The default branch is persistently red |

Nothing outside this list may ever be graded 🔴 — a check not on it tops out at 🟡.

## Capped dimensions

**Dimension 5 (Task surface)** grades *invitation*, not *possibility*. A contributor who brings their own itch can contribute to a repo with zero `good first issue`s. An empty task surface caps inbound volume; it doesn't close the door.

**Dimension 10 (Review & release loop)** is the only dimension graded from **people's behaviour** rather than artifacts. Turning a median into a blocking verdict is a judgment about a team, made from a sample of a dozen PRs, and it's the failure mode most likely to make a maintainer discard the whole report. It stays a measured fact, not a gate.

The cap is not a way to hide a dead project. Compensating rule: **when any 10.x check is 🟡, the report's lead paragraph must state the stewardship numbers first** — median time to first review, external PRs merged, oldest unanswered PR — before anything else. A 🟡 verdict on a repo nobody reviews should read as uncomfortable as it is; the audit never lets the cap soften that.

## Conservatism rule

A grade other than 🟢 must cite a path, a line, a command with its output, a count, or a documented claim shown false. If you can't produce that, the check is 🟢 (you checked and it's fine) or ⚪ (you couldn't check) — never 🟡 on a hunch, never 🟢 on something you didn't check.

**Addition specific to this skill:** absence is only a finding when the project is the kind of project that needs the thing. No `CHANGELOG.md` in a repo that's never released, no migration convention with no database, no screenshot in a headless library, no `GOVERNANCE.md` for a project that says in one sentence it has one maintainer: each is 🟢 or n/a, never 🟡. A 🟡 must cite the thing the project *has* that makes the gap real ("an extension registry at `src/plugins/index.ts:12` that no doc mentions"), never just "no plugin docs."

## Boundary with loop-engineering-audit

Both skills read a repo and produce an ordered plan, and they ask different questions on behalf of different readers. `loop-engineering-audit` asks whether an **unattended agent** can take a defined task, verify it with local checks, and hand off a PR with no human in the inner loop — it owns agent-facing context (`AGENTS.md`/`CLAUDE.md`), non-interactive command shape and wall time, guardrails that make unsupervised mistakes cheap, parallel-worktree isolation, and the advisory automation map. `audit-open-source-repo` asks whether an **outside human** who has never seen the codebase can discover the project, set it up without the team's secrets, find scoped work, build a feature to the project's own quality bar, and get it reviewed, merged and released — it owns README/LICENSE/CONTRIBUTING/governance, the fork-based path, machine-enforced guidelines, orientation docs for feature work, the task surface, and the review and release loop measured from history. Where they touch the same artifact they grade a different property of it: CI is "does the loop have a finish line" there and "does the gate run and pass on a fork PR with no secrets" here; tests are "is a green suite a safety net for an unsupervised change" there and "can a stranger run them and know what to write" here; conventions are "can an agent read them" there and "are they enforced by a command so a reviewer never re-teaches them" here. Run both on a repo that wants both audiences — never merge the reports, never cross-cite a check id, and never let one skill's plan absorb the other's rows.
