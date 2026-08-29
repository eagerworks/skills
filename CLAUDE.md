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
evals/<name>/       # NOT shipped — repo-level test harness
  evals.json        # question/answer + expectations pairs that verify skill quality
```

The boundary is the rule: **anything users should receive goes inside `skills/<name>/`; anything purely for developing or validating the skill stays out of it.** `evals/<name>/` mirrors the skill name. The skills.sh CLI ships the entire directory containing a `SKILL.md`, which is why evals must live outside it.

### Progressive disclosure (the core design principle)

`SKILL.md` is loaded up front, so it must stay **lean** — it's a hub: when-to-use frontmatter, a version-detection step, a table pointing to `references/*.md`, and critical gotchas. Depth belongs in `references/*.md`, which the agent opens only when relevant. When editing, push detail down into `references/` rather than growing `SKILL.md`.

## The skills

**kamal** deploys Dockerized apps with Kamal. **Version-aware**: defaults to **Kamal 2.x**; all Kamal 1.9.x content lives *exclusively* in `references/kamal-v1.md`. `SKILL.md` opens with a version-detection step (`kamal version`, or infer from `traefik:`/`.env` → v1 vs `proxy:`/`.kamal/secrets` → v2). When adding version-specific examples, mark them (`# Kamal 2.x only` / `# Kamal 1.x only`) and never mix v1 syntax into the v2 references.

**loop-engineering-audit** audits a repository for readiness to be developed through autonomous agent loops ("loop engineering") across seven dimensions — agent-facing context, reproducible environment, fast deterministic verification, test coverage, task definition surface, CI & merge gates, guardrails — graded 🔴 Blocker / 🟡 Gap / 🟢 Ready / ⚪ Unverifiable, rolled up to a mechanical verdict and an ordered **Work Plan**. `references/rubric.md` is the single source of truth for checks and grades; `SKILL.md` only summarizes it. It has exactly **one write**: the report is printed in full in chat and saved to `docs/loop-engineering-audit.md` in the audited repo (fixed name, overwritten, never committed — see `docs/decision-records/2026-08-28--audit-report-saved-to-docs.md`). It may execute the project's lint/typecheck/test/build once to measure them, never setup/migrate/install/deploy commands (`references/audit-workflow.md`).

**ui-ux-audit** audits a project's user interface and experience across ten dimensions — information architecture & navigation, visual consistency & design system, layout & responsiveness, accessibility (WCAG 2.2 AA), forms & input, feedback & system states, content & microcopy, performance as experienced, flow simplicity, product instrumentation & metrics — graded 🔴 Blocker / 🟡 Gap / 🟢 Solid / ⚪ Unverifiable, rolled up to a mechanical verdict (Needs work / Acceptable with gaps / Solid) and an ordered **Work Plan**. `references/rubric.md` is the single source of truth for checks, grades, and the **recommendation rule**: every check — 🟢 and ⚪ included — ends with a `→ Recommendation:` line (fix / Keep / Verify); `SKILL.md` only summarizes it. Dimension 9 (`references/flow-simplicity.md`) grades only against a **flow walk** and its recommendation is the proposed step sequence, never "simplify"; dimension 10 (`references/instrumentation.md`) is capped at 🟡 except PII in event payloads, its recommendation is an **Instrumentation plan** (flow → events → metric → decision → candidate tool), tools are always dated candidates plus a mandatory "survey the current market" line, at most two web searches may confirm them (`marketCheck`), and the audit never installs or configures a tool (see `docs/decision-records/2026-08-29--ui-ux-audit-flow-and-instrumentation-dimensions.md`). Graded from source; an optional, strictly bounded browser pass (navigate, resize, tab, screenshot) runs only against an app that is already running — it never starts a server, installs, submits real forms, or clicks destructive actions (`references/audit-workflow.md`). Exactly **one write**: the report is printed in full in chat and saved to `docs/ui-ux-audit.md` in the audited repo (fixed name, overwritten, never committed — see `docs/decision-records/2026-08-29--ui-ux-audit-report-saved-to-docs.md`).

**pr-review** reviews a diff (branch, PR, staged, or working-tree changes) against a fixed five-lens rubric — correctness, security & data integrity, repo-convention conformance, test coverage, documentation & decision capture — ported from the `dizenz/agent-skills` `code-reviewer` subagent and its review rubric, generalized to be plugin-free and stack-agnostic (Rails + Node/TypeScript). `references/rubric.md` is the single source of truth for the rubric and severity ladder; `SKILL.md` only summarizes it — never duplicate rubric detail back into `SKILL.md`. Read-only by default: the standard review never edits, commits, or pushes; the optional fix loop only runs when a user explicitly asks for it (see `references/workflow.md`). The one automatic mutation is posting the report as a comment on the PR under review (`gh pr comment`) when the scope is a GitHub PR — the same markdown report the console shows, opt-out via `review.postToPr: false`; if `gh` isn't installed/authenticated the skill offers to post once it is, never fails the review. The documentation lens (on by default, switchable off via `.eagerworks/pr-review.json`) turns a diff that makes an existing doc false into a normal severity-rated finding, and an undocumented non-obvious decision into a separate, capped, non-blocking suggestion — never a merge blocker, never fabricated rationale.

## Authoring conventions (from CONTRIBUTING.md)

- **Good/bad config**: use `✅ correct` / `❌ wrong` to contrast valid and invalid config.
- **No real secrets**: placeholders only (`your-token`, `ghcr.io/your-org/your-app`, `192.168.0.1`, `your.domain.com`). The skill teaches secret hygiene — examples must model it.
- **Code blocks for everything**: fence all commands/config with the right language tag (`bash`, `yaml`, `ruby`).
- Keep `SKILL.md`'s frontmatter `description` specific about *when* to use the skill — agents match against it.

## Adding a new skill

1. `skills/<name>/SKILL.md` with `name` + `description` frontmatter.
2. `references/` and `assets/` inside `skills/<name>/`; keep `SKILL.md` lean.
3. `skills/<name>/README.md` human overview.
4. `evals/<name>/evals.json` with cases.
5. Add a row to the **Available skills** table in `README.md`.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/), matching existing history:
- `docs:` — content changes to `SKILL.md`, `references/`, `assets/`
- `feat:` — new reference files, new asset templates, significant new coverage
- `fix:` — corrections to wrong/outdated information
- `chore:` — eval cases, project-level housekeeping

## skill-creator (vendored tooling)

`.agents/skills/skill-creator/` is a vendored skill (tracked in `skills-lock.json`, sourced from `anthropics/skills`) used to author/eval/optimize skills. It contains the only executable code in the repo (Python under `scripts/` and `eval-viewer/`). Don't hand-edit it as project content — it's a managed dependency.
