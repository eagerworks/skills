# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`eagerworks/skills` is a **collection of portable markdown agent skills** — not a runnable library. There is no build, lint, or compile step. The "product" is documentation that teaches coding agents (Claude Code, Cursor, Copilot, Codex, Amp, any [skills.sh](https://www.skills.sh)-compatible tool) how to do a specific job well. Contributions are almost always edits to markdown files.

## Architecture

Two parallel trees, intentionally kept separate (see `docs/decision-records/2026-06-30--evals-separate-from-skills.md`):

```
skills/<name>/      # SHIPPED to users — the skills.sh CLI copies this whole dir
  SKILL.md          # agent entrypoint: frontmatter `description` (match trigger) + lean hub
  references/*.md    # in-depth docs, loaded by the agent ON DEMAND
  assets/           # copyable starter files / templates (e.g. deploy.yml)
  README.md         # human-facing overview
  CHANGELOG.md      # per-version history: what changed + what config/files a consumer must touch
evals/<name>/       # NOT shipped — repo-level test harness
  evals.json        # question/answer + expectations pairs that verify skill quality
```

The boundary is the rule: **anything users should receive goes inside `skills/<name>/`; anything purely for developing or validating the skill stays out of it.** `evals/<name>/` mirrors the skill name. The skills.sh CLI ships the entire directory containing a `SKILL.md`, which is why evals must live outside it.

### Progressive disclosure (the core design principle)

`SKILL.md` is loaded up front, so it must stay **lean** — it's a hub: when-to-use frontmatter, a version-detection step, a table pointing to `references/*.md`, and critical gotchas. Depth belongs in `references/*.md`, which the agent opens only when relevant. When editing, push detail down into `references/` rather than growing `SKILL.md`.

## The skills

**commit** commits everything modified or new in the working tree by default, grouped into a series of Conventional Commits instead of one commit for everything pending — `commit.includeUntracked` defaults to `"always"`; a repo that wants a confirmation step for new files instead sets it to `"ask"` (or `"never"` to exclude them entirely) in `.eagerworks/commit.json`. The unit is the whole file, never a hunk (`git add -p` is interactive and unavailable to an agent); `references/grouping.md` is the single source of truth for the grouping ladder — an already-staged index first, then an explicit instruction, then change intent, then mechanical companions (lockfile/manifest, migration/schema), then everything left over on its own — plus the default dependency-first ordering and the hard exclusion list (`.env`, credentials, `node_modules`, build output, screenshots), always disclosed when it triggers, never silent, regardless of the `includeUntracked` setting. The type/scope vocabulary is inferred from the repo's own `git log` before falling back to the full Conventional Commits type set, overridable via `.eagerworks/commit.json`. It stops at the commit: never `git push`, `--amend`, `rebase`, `reset --hard`, or branch creation — see `docs/decision-records/2026-09-07--create-pr-write-posture.md` for the write-posture precedent it inherits. `create-pr` hands off to this skill for the commit step (`skills/create-pr/references/workflow.md` → "Commit anything pending").

**create-pr** opens or updates a pull request for the current branch: resolves the base branch from evidence (never assumes the repo's default branch), writes a description with a fixed structure and no hand-wrapped paragraphs, derives the verification checklist from tooling actually detected in the repo, splits `Test plan` into `Automated` results and tickable `Manual` test scenarios (setup, per-step checkboxes, an expected-result step) for whoever reviews the PR's quality — on by default via `pr.manualTestCases`, uncapped unless a repo opts into `maxScenarios` (over which scenarios are prioritized by risk), and never fabricating a scenario, credential, or step the diff doesn't support (an unknown becomes `TODO(author):`, and a change with no observable surface gets a one-line reason instead of the subsection) — and attaches screenshots via the GitHub CLI's `--attach` flag (`gh` ≥ 2.99.0) when the change is UI-visible — never fabricating an image URL or committing a screenshot into the repo. Unlike the read-only review skills, it mutates by design (commits pending work, pushes, creates/edits the PR) as a deliberate, narrowly-scoped exception to this repo's read-only-by-default posture — see `docs/decision-records/2026-09-07--create-pr-write-posture.md`. PR title/description language is configured via `pr.language` (English by default), never inferred from the conversation's language.

**decision-record** writes or updates an architecture decision record (ADR) as a dated markdown file under `docs/decision-records/` — a stack-agnostic, repo-agnostic format: `YYYY-MM-DD--kebab-slug.md`, a one-sentence declarative title with no leading number and no `Status` field, and exactly four `##` sections (Context, Decision, Consequences, Related), applied the same way regardless of what convention the target repo already uses (it says so out loud rather than silently blending in — `references/format.md` → "When the repo already has records"). Carries no config file; the format is fixed on purpose. Its one write is the record itself, committed with a plain `docs:` commit — it never touches the code the record is about and never opens a PR. `references/format.md` and `references/writing.md` are the source of truth; `SKILL.md` only summarizes them.

**kamal** deploys Dockerized apps with Kamal. **Version-aware**: defaults to **Kamal 2.x**; all Kamal 1.9.x content lives *exclusively* in `references/kamal-v1.md`. `SKILL.md` opens with a version-detection step (`kamal version`, or infer from `traefik:`/`.env` → v1 vs `proxy:`/`.kamal/secrets` → v2). When adding version-specific examples, mark them (`# Kamal 2.x only` / `# Kamal 1.x only`) and never mix v1 syntax into the v2 references.

**loop-engineering-audit** audits a repository for readiness to be developed through autonomous agent loops ("loop engineering") across eight dimensions — agent-facing context, reproducible environment, fast deterministic verification, test coverage, task definition surface, CI & merge gates, guardrails, and parallel-session readiness (can several agent sessions work the repo at once, in separate `git worktree`s, without colliding) — graded 🔴 Blocker / 🟡 Gap / 🟢 Ready / ⚪ Unverifiable, rolled up to a mechanical verdict and an ordered **Work Plan**. No parallel-session check is ever 🔴 (it caps throughput, not the verdict), and the audit never runs `git worktree add` — that would be a second write. `references/rubric.md` is the single source of truth for checks and grades; `SKILL.md` only summarizes it. It has exactly **one write**: the report is printed in full in chat and saved to `docs/loop-engineering-audit.md` in the audited repo (fixed name, overwritten, never committed — see `docs/decision-records/2026-08-28--audit-report-saved-to-docs.md`). It may execute the project's lint/typecheck/test/build once to measure them, never setup/migrate/install/deploy commands (`references/audit-workflow.md`).

Beyond readiness it also answers "what should we automate here" as strictly **advisory** output: an automation map classifying ten delivery stages (Automated / Assisted / Manual / Human by design / Absent — plain words, deliberately not a fifth grade, with "Human by design" so the map can say a stage should stay human) and up to five recommended agent loops from a catalog of ten, each with a trigger, a machine-checkable done signal, prerequisites expressed as rubric check ids, a blast-radius risk level (L1 Contained / L2 Shared-state / L3 External-effect — defined by reversibility, never by how likely the agent is to be wrong), guardrails, what to configure, and an owner/cadence/kill-switch line. `references/loop-catalog.md` is the single source of truth for that concern and adds no checks to `references/rubric.md`. The hard line: loop recommendations add no Work Plan rows, change no grade, and never move the verdict — a blocked loop cites the Work Plan item numbers that unblock it and that pointer only runs one way (see `docs/decision-records/2026-09-12--loop-recommendations-are-advisory.md` and `docs/decision-records/2026-09-12--parallel-session-readiness-is-a-capped-dimension.md`). The skill still writes exactly one file: it never creates a workflow, a permissions file, or a playbook. Configured via `loops.{enabled,maxRecommended,maxRiskLevel}` and `dimensions.parallelSessions`, with the same footer-disclosure discipline as the other dimensions.

**pr-review** reviews a diff (branch, PR, staged, or working-tree changes) against a fixed five-lens rubric — correctness, security & data integrity, repo-convention conformance, test coverage, documentation & decision capture — ported from the `dizenz/agent-skills` `code-reviewer` subagent and its review rubric, generalized to be plugin-free and stack-agnostic (Rails + Node/TypeScript). `references/rubric.md` is the single source of truth for the rubric and severity ladder; `SKILL.md` only summarizes it — never duplicate rubric detail back into `SKILL.md`. Read-only by default: the standard review never edits, commits, or pushes; the optional fix loop only runs when a user explicitly asks for it (see `references/workflow.md`). The one automatic mutation is posting the report to the PR under review when the scope is a GitHub PR — by default a `gh api` review with inline comments on each finding's line plus a summary body, falling back to a single `gh pr comment` when the inline post fails or when `review.commentStyle: "summary"` is set; opt out of posting entirely via `review.postToPr: false`; if `gh` isn't installed/authenticated the skill offers to post once it is, never fails the review. The documentation lens (on by default, switchable off via `.eagerworks/pr-review.json`) turns a diff that makes an existing doc false into a normal severity-rated finding, and an undocumented non-obvious decision into a separate, capped, non-blocking suggestion — never a merge blocker, never fabricated rationale. The report (console output, inline comments, summary body, disclosure lines) is written in English by default, set per repo via `review.language` — the language of the conversation is never an input to that choice, so the report stays deterministic regardless of what language the user is chatting in; the `### FINDINGS`/`### DOCUMENTATION` machine-parseable block's keys and severity enum are never translated.

**repo-handoff** prepares the handoff of a codebase inherited from another team. It reads the repo from the receiving team's point of view across ten dimensions (overview & architecture, environment & setup, build/test/quality, infrastructure & deploy, data, third-party services & credentials, security & access, code health & debt, process & history, operations & support), grades each check 🔴 Missing / 🟡 Partial / 🟢 Documented / ⚪ Unverifiable, rolls up to Blocked / At risk / Ready, and — the centrepiece — emits a numbered, P0/P1/P2-prioritized list of **questions for the previous team**, each citing the evidence gap and carrying an `Answer:` line (see `docs/decision-records/2026-08-29--repo-handoff-questions-as-first-class-output.md`). `references/dimensions.md` is the single source of truth for checks and the question bank; `SKILL.md` only summarizes it. Exactly **one write**: the report is printed in full in chat and saved to `docs/repo-handoff.md` in the analyzed repo (fixed name, overwritten on re-run, previously filled answers carried over, never committed). It may run lint/typecheck/test and read-only dependency audits once, never setup/install/migrate/deploy, and never prints secret values — only locations and names.

**audit-soc2** plans and runs a SOC 2 readiness effort. Two behaviors define it and must not be diluted when editing: it **asks a batched intake interview before recommending anything** (`SKILL.md`'s "The Intake Interview" section — never dump the full question bank at once), and it **always outputs a fixed plan shape** (scope statement → gap matrix → five-phase roadmap → top-5 priorities → stated assumptions, per "Producing the Plan"). It defaults to a **DIY-first** tooling stance — no assumption of a compliance platform (Vanta/Drata/Secureframe); `references/tooling.md` covers those only as a closing "when it's worth it" note. `SKILL.md` opens with an engagement-framing step (own company vs. client engagement; Type I vs. Type II; current stage) analogous to kamal's version detection.

## Authoring conventions (from CONTRIBUTING.md)

- **Good/bad config**: use `✅ correct` / `❌ wrong` to contrast valid and invalid config.
- **No real secrets**: placeholders only (`your-token`, `ghcr.io/your-org/your-app`, `192.168.0.1`, `your.domain.com`). The skill teaches secret hygiene — examples must model it.
- **Code blocks for everything**: fence all commands/config with the right language tag (`bash`, `yaml`, `ruby`).
- Keep `SKILL.md`'s frontmatter `description` specific about *when* to use the skill — agents match against it.

## Versioning & changelogs

Every skill carries `metadata.version` (semver) in `SKILL.md`'s frontmatter and a `skills/<name>/CHANGELOG.md` — shipped, unlike `evals/`, because it's the only place a consuming repo can see what changed between the version it has and the version it's pulling. Bump the version whenever a change to `SKILL.md`, `references/`, or `assets/` would be visible to a consumer, and add a changelog entry in the same commit:

- **patch** — a correction with no behavior or config change (wrong command, outdated flag, typo).
- **minor** — new optional behavior, a new optional config key, new reference coverage — backward compatible, nothing existing breaks.
- **major** — a config key renamed/removed, a default flipped, or a new file the consumer must create for the skill to keep working.

Every entry that touches config or requires creating/editing a file in the consumer's repo says so explicitly under a **Config** (or **Migration**) heading — never leave it to be inferred from the diff. This is what `references/config.md` documents in schema form; the changelog documents it in "what changed and why you'd notice" form. See any existing `skills/*/CHANGELOG.md` for the format.

## Adding a new skill

1. `skills/<name>/SKILL.md` with `name` + `description` frontmatter, plus `metadata.version: "1.0.0"`.
2. `references/` and `assets/` inside `skills/<name>/`; keep `SKILL.md` lean.
3. `skills/<name>/README.md` human overview.
4. `skills/<name>/CHANGELOG.md` with an initial `[1.0.0]` entry.
5. `evals/<name>/evals.json` with cases.
6. Add a row to the **Available skills** table in `README.md`.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/), matching existing history:
- `docs:` — content changes to `SKILL.md`, `references/`, `assets/`
- `feat:` — new reference files, new asset templates, significant new coverage
- `fix:` — corrections to wrong/outdated information
- `chore:` — eval cases, project-level housekeeping

## skill-creator (vendored tooling)

`.agents/skills/skill-creator/` is a vendored skill (tracked in `skills-lock.json`, sourced from `anthropics/skills`) used to author/eval/optimize skills. It contains the only executable code in the repo (Python under `scripts/` and `eval-viewer/`). Don't hand-edit it as project content — it's a managed dependency.
