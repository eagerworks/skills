# skills

Agent skills maintained by [Eagerworks](https://eagerworks.com) — portable, progressive-disclosure knowledge bases that teach coding agents how to do a specific job well.

Each skill is plain markdown and works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic tool that reads the [Agent Skills](https://www.skills.sh) format. Install them with one command via the [skills.sh CLI](https://www.skills.sh).

## Available skills

### Git workflow & review

| [commit](skills/commit/) |
|:---|
| Commits everything modified or new in the working tree by default, grouped into a series of coherent [Conventional Commits](https://www.conventionalcommits.org/) — one commit per logical change, staged by explicit path (never `git add -A`, never the interactive `git add -p`) — with the type/scope vocabulary inferred from the repo's own git history rather than assuming the full spec is in use, an exclusion list for secrets/build output/screenshots that's always disclosed when it triggers, and honest handling of pre-commit hooks (a rejection stops the series, a formatter rewrite gets one retry, never `--no-verify`). Configurable per repo to ask before including new files instead. Stops at the commit — pushing and opening a PR is `create-pr`'s job. |

| [create-pr](skills/create-pr/) |
|:---|
| Opens or updates a pull request: base-branch resolution from evidence, a description structure with no hand-wrapped paragraphs, a verification checklist derived from tooling actually detected in the repo, a `Test plan` split into `Automated` results and tickable `Manual` test scenarios (setup, steps, expected result) for whoever reviews the PR's quality, and screenshots attached via the GitHub CLI's `--attach` flag (`gh` ≥ 2.99.0) — the first officially supported way to get an image into a PR body via automation — with an explicit placeholder instead of a fabricated URL or test step when none is available, and the repo's own PR template resolved by asking rather than assuming either side. |

| [pr-review](skills/pr-review/) |
|:---|
| Code review for Rails and Node/TypeScript diffs: correctness, security & multi-tenant scoping, repo-convention conformance, test coverage against acceptance criteria, and documentation & decision capture (stale docs, undocumented decisions), with a conservative severity ladder, markdown or machine-parseable output, and an optional review-fix loop. When reviewing a GitHub PR, it also posts the report on the PR as a comment, in English by default — configurable per repo via `review.language`, regardless of what language the conversation is in. |

### Documentation

| [decision-record](skills/decision-record/) |
|:---|
| Writes or updates an architecture decision record (ADR) as a dated markdown file under `docs/decision-records/`: a one-sentence declarative title with no leading number and no `Status` field, and exactly four sections — Context, Decision, Consequences, Related — applied the same way in every repo. Requires the failure mode on both sides of the choice in Context, never invents the "why" (asks or leaves a `TODO(author):` instead), and treats a changed decision as a new record that links back rather than a rewrite of the old one. |

### API design

| [rest-api-design](skills/rest-api-design/) |
|:---|
| Design and review REST APIs: resource modeling, HTTP methods & status codes, payload and RFC 9457 error shapes, pagination/filtering, versioning & deprecation, auth, rate limiting, security pitfalls, and OpenAPI 3.1 — with an existing-API survey step so new endpoints match the conventions already in the codebase. |

### Deployment

| [kamal](skills/kamal/) |
|:---|
| Zero-downtime [Kamal](https://kamal-deploy.org) deployments (v2.x + 1.9.x): first-time setup, deploys, rollbacks, rolling deploys, `kamal-proxy` + Let's Encrypt SSL, secrets & vault adapters, accessories, builders/multiarch, troubleshooting, and the v1→v2 upgrade. |

### Audits & compliance

| [audit-hipaa](skills/audit-hipaa/) |
|:---|
| Audits a codebase and its infrastructure config against the [HIPAA Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html): locates PHI in data models, logs, error trackers, analytics, and outbound LLM/API calls, checks §164.312 technical safeguards (access control, audit controls, integrity, authentication, transmission security), and routes BAA/administrative obligations to a human — output as a severity-graded audit report written to a dated file in the repo. |

| [audit-soc2](skills/audit-soc2/) |
|:---|
| Plans and runs a SOC 2 readiness effort: a structured intake interview, Trust Services Criteria scoping (Type I/II, system boundary, subservice orgs), a gap analysis and phased roadmap, required policy skeletons, DIY evidence collection, and the CPA audit process. |

| [mobile-store-review](skills/mobile-store-review/) |
|:---|
| Audits an Expo/React Native or native mobile app — standalone or inside a Turborepo — against the [Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) and [Google Play Developer Program Policies](https://play.google.com/about/developer-content-policy/): permissions & usage descriptions, privacy manifests & App Tracking Transparency, App Privacy vs. Data safety, account deletion, IAP & external payments, SDK/target-API floors, versioning & credentials, and EAS/monorepo build config — output as a severity-graded audit report. |

| [loop-engineering-audit](skills/loop-engineering-audit/) |
|:---|
| Audits a repository for loop-engineering readiness — whether an AI coding agent can take a task, implement it, verify it with the project's own checks, and hand off a PR unattended: agent-facing docs, reproducible setup, fast non-interactive verification, test safety net, task/PR conventions, CI gates, guardrails, and parallel-session readiness (worktrees that don't collide) — graded 🔴/🟡/🟢/⚪ with an ordered Work Plan, printed in chat and saved to `docs/loop-engineering-audit.md`. Then, as advisory output that never enters the Work Plan or the verdict: a map of which delivery stages are Automated / Assisted / Manual / Human by design / Absent, and up to five concrete agent loops the project could actually create and maintain — each with its blast-radius risk level (L1 contained → L3 external-effect), the prerequisites and guardrails it needs, and what to configure to stand it up. |

| [repo-handoff](skills/repo-handoff/) |
|:---|
| Prepares the handoff of a codebase inherited from another team: reads architecture, setup, build/test, infrastructure & deploy, data, third-party services & credentials, security & access, code health, process/history, and operations — graded 🔴/🟡/🟢/⚪ — and turns every gap into a prioritized (P0/P1/P2) list of **questions for the previous team** plus an access & ownership transfer checklist, printed in chat and saved to `docs/repo-handoff.md` (answers preserved across re-runs). |

## Install

Install with the [skills.sh CLI](https://www.skills.sh) — one command, no manual file copying or pointer files to write. It works with Claude Code and 70+ other agents (Cursor, GitHub Copilot, Codex, Amp, and more).

```bash
# install a specific skill
npx skills add eagerworks/skills --skill kamal

# or pick interactively from the collection
npx skills add eagerworks/skills

# or install every skill in the collection
npx skills add eagerworks/skills --all
```

A few notes:

- **Agent detection** — the CLI auto-detects the coding agents you have installed. If none are detected, it prompts you to pick which ones to install to.
- **Target specific tools** with `-a` / `--agent` (repeatable):

  ```bash
  npx skills add eagerworks/skills --skill kamal -a claude-code -a cursor
  ```

  Supported slugs include `claude-code`, `cursor`, `github-copilot`, `codex`, and `amp` (plus dozens more).
- **Global vs project** — installs into your project by default. Add `-g` / `--global` to make it available in every project.
- **No pointer files** — agents with native Agent Skills support load each skill's `SKILL.md` automatically from its frontmatter `description`, and open `references/*.md` on demand. Nothing else to wire up.

> **Known issue with Claude Code** — `npx skills add` (even with `-a claude-code`, global or project-scoped) can install the skill into `.agents/skills/<name>/` and record it in `skills-lock.json`, without creating the symlink in `.claude/skills/<name>/` that Claude Code actually reads from. If a freshly installed skill doesn't show up, check whether that symlink exists; if it doesn't, create it yourself and reload:
>
> ```bash
> # project-level
> ln -s ../../.agents/skills/<name> .claude/skills/<name>
> # global
> ln -s ~/.agents/skills/<name> ~/.claude/skills/<name>
> ```
>
> Then run `/reload-skills` inside Claude Code so the session picks it up. Tracked upstream: [vercel-labs/skills#744](https://github.com/vercel-labs/skills/issues/744), [#851](https://github.com/vercel-labs/skills/issues/851), [#1355](https://github.com/vercel-labs/skills/issues/1355).

<details>
<summary><strong>Manual install (advanced)</strong> — for tools without skills.sh support, or if you prefer to vendor the files yourself</summary>

The universal pattern is the same for every tool, and every skill lives under `skills/<name>/`:

1. **Vendor the skill** into your project (or a global location).
2. **Add a small pointer** in your tool's rules/instructions file so the agent knows to load the skill's `SKILL.md` when it's relevant.

The agent opens `references/*.md` files on demand, so the pointer stays tiny while the full knowledge base is always available. All integrations point at the same `SKILL.md` and `references/` files, so updating the vendored copy updates every integration at once.

The examples below use the **kamal** skill — swap `kamal` for any other skill in the collection.

### Claude Code

**Global install** (available in every project):

```bash
# copy:
cp -r /path/to/skills/skills/kamal ~/.claude/skills/kamal

# or symlink (changes in the repo reflect immediately):
ln -s /path/to/skills/skills/kamal ~/.claude/skills/kamal
```

**Project-level install** (committed to the repo, shared with your team):

```bash
mkdir -p .claude/skills
cp -r /path/to/skills/skills/kamal .claude/skills/kamal
# or: ln -s /path/to/skills/skills/kamal .claude/skills/kamal
```

Claude Code reads the `name` and `description` frontmatter in `SKILL.md` and loads the skill automatically — no extra pointer file needed.

### Cursor

First, vendor the collection into your project:

```bash
git submodule add https://github.com/eagerworks/skills tools/eagerworks-skills
# or: cp -r /path/to/skills tools/eagerworks-skills
```

Then create `.cursor/rules/kamal.mdc`:

```markdown
---
description: >
  Expert guide for setting up and managing Kamal deployments — the Basecamp tool for zero-downtime
  container deployments to bare-metal servers and VMs. Use this skill whenever the user: mentions
  Kamal, kamal-proxy, or Traefik-based deployments; wants to deploy a Dockerized app to a VPS, VM,
  or bare-metal server; is writing or editing a config/deploy.yml or .kamal/secrets file; asks about
  zero-downtime deploys, rolling deploys, rollbacks, or deploy hooks; needs to set up a Docker
  registry, builder, or SSH connection for deployments; asks about Kamal accessories (databases,
  Redis, etc.); is troubleshooting a failed deploy, healthcheck error, or lock issue; is upgrading
  from Kamal 1 to Kamal 2; or wants to run commands inside a running container. Also use when
  the user says things like "deploy my app without downtime", "deploy to my own server", or
  "set up SSL for my self-hosted app" — even if they don't name Kamal.
alwaysApply: false
---

Read @tools/eagerworks-skills/skills/kamal/SKILL.md for Kamal deployment guidance.
Load the matching file from @tools/eagerworks-skills/skills/kamal/references/ on demand as needed.
```

Cursor matches the `description` to incoming requests and loads the rule automatically — no need to mention Kamal explicitly.

### GitHub Copilot

Vendor the collection first (same as above):

```bash
git submodule add https://github.com/eagerworks/skills tools/eagerworks-skills
# or: cp -r /path/to/skills tools/eagerworks-skills
```

**Option A — always-on** (applies to every file): append to `.github/copilot-instructions.md`:

```markdown
## Kamal deployments

When helping with Kamal deployments, read `tools/eagerworks-skills/skills/kamal/SKILL.md` for version detection,
setup workflow, a command cheatsheet, and critical gotchas. Load the relevant file from
`tools/eagerworks-skills/skills/kamal/references/` on demand (configuration, commands, workflows, secrets-and-hooks,
proxy-and-ssl, builders, troubleshooting, kamal-v1).
```

**Option B — path-scoped** (triggers only for Kamal files): create `.github/instructions/kamal.instructions.md`:

```markdown
---
applyTo: "**/deploy.yml,**/.kamal/**,**/kamal/**"
---

When helping with Kamal deployments, read `tools/eagerworks-skills/skills/kamal/SKILL.md` for version detection,
setup workflow, a command cheatsheet, and critical gotchas. Load the relevant file from
`tools/eagerworks-skills/skills/kamal/references/` on demand (configuration, commands, workflows, secrets-and-hooks,
proxy-and-ssl, builders, troubleshooting, kamal-v1).
```

### Codex, Amp, and other AGENTS.md-compatible tools

Vendor the collection (same as above), then add a section to your project's `AGENTS.md`:

```markdown
## Kamal deployments

When helping with Kamal deployments, read `tools/eagerworks-skills/skills/kamal/SKILL.md` for version detection,
setup workflow, a command cheatsheet, and critical gotchas. Load the relevant file from
`tools/eagerworks-skills/skills/kamal/references/` on demand (configuration, commands, workflows, secrets-and-hooks,
proxy-and-ssl, builders, troubleshooting, kamal-v1).
```

</details>

### Any LLM — Claude.ai, API, or standalone

The content is plain markdown and works in any interface:

| Interface | How to use |
|---|---|
| **Claude.ai** | Paste a skill's `SKILL.md` + the relevant `references/*.md` into your conversation |
| **Claude API** | Include the files as system-prompt context |
| **Any LLM / standalone docs** | The `references/` files are plain documentation — use them directly |

## Repository structure

Each skill is a self-contained directory under `skills/<name>/` — this is the boundary the skills.sh CLI ships, so everything the agent needs lives inside it:

```
skills/
  kamal/                        # one skill (what skills.sh installs)
    SKILL.md                    # agent entrypoint: when to use it + a lean hub
    references/                 # in-depth docs, loaded on demand
    assets/                     # copyable starter files / templates
    README.md                   # human-facing overview of the skill
    CHANGELOG.md                # per-version history + what config/files to update
  audit-soc2/                   # another skill, same shape
    SKILL.md
    references/
    assets/
    README.md
    CHANGELOG.md
evals/
  kamal/
    evals.json                  # per-skill eval cases (repo-level harness, not shipped)
  audit-soc2/
    evals.json
```

## Adding a new skill

1. Create `skills/<name>/SKILL.md` with `name` and `description` frontmatter (the `description` is what agents match against — make it specific about when to use the skill), plus `metadata.version: "1.0.0"`.
2. Add `references/` and `assets/` inside `skills/<name>/` as needed; keep `SKILL.md` lean and push depth into `references/`.
3. Add `skills/<name>/CHANGELOG.md` with an initial `[1.0.0]` entry.
4. Add eval cases at `evals/<name>/evals.json`.
5. Add a row to the **Available skills** table above.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for authoring conventions.

## How it stays in sync

Every tool reads the same `SKILL.md` and `references/` files. Re-run `npx skills add eagerworks/skills --skill <name>` to pull the latest version (or, on a manual install, update your vendored copy) and every agent picks up the changes — there is no per-tool content to keep in sync.

Each skill also carries a `CHANGELOG.md` and a semver `metadata.version` in `SKILL.md`'s frontmatter. Before or after updating, check the skill's `CHANGELOG.md` (in your vendored copy, or on GitHub) for anything landed between the version you had and the one you're pulling — entries call out, under a **Config** heading, any `.eagerworks/<name>.json` key or other file you need to add or change for the new behavior to apply.

The progressive-disclosure design means the agent loads only `SKILL.md` up front, while the full knowledge base is always available to open on demand.

## License

MIT © [Eagerworks](https://eagerworks.com)
