# architecture-design

A portable agent skill that guides you through defining the **v1 architecture of a new system**. It runs a phased interview — problem, users and scale, scope, constraints, quality attributes, data and integrations, delivery — one phase at a time with sensible defaults, turns the answers into ranked drivers, proposes the simplest architecture that satisfies them with alternatives and rationale, and writes the result to `docs/architecture.md` in your project. Stack-agnostic. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Discovery first: reads the repo (manifests, README, CI, existing `docs/architecture.md`) so it never asks what the code already answers
- A seven-phase interview with at most four questions per turn, each with a default you can accept with "yes", plus conditional follow-ups (multi-tenancy, real-time, payments, compliance, offline, AI features…)
- A gate that says when there is enough information to propose — and how to proceed on assumptions when the user says "just propose something"
- Decision heuristics: modular monolith by default, one relational database, a job queue, managed hosting; anything more complex needs a named driver
- A short proposal in chat with one round of iteration before the document is written; disagreements are recorded as the user's decision with their reason, not relitigated
- The document: goals, non-goals, drivers, quality attributes, mermaid architecture/ER/sequence diagrams, modules, data and storage, integrations, infrastructure, cross-cutting concerns, a decision log with alternatives, risks and open questions, evolution path with triggers
- The document is printed in chat **and** saved to `docs/architecture.md` (the directory is created if needed) — the one file the skill writes; it never scaffolds code, commits, or pushes
- Revise mode: when a doc already exists, asks only about what changed and keeps the decision log with superseded rows

## Layout

```
SKILL.md                    # hub: discovery, the seven phases, synthesis, gotchas (agent entrypoint)
references/
  interview.md              # questions per phase, follow-ups, skip rules, defaults, "enough to propose" gate
  design-workflow.md        # phase-by-phase procedure: discovery, interview, synthesis, proposal, delivery, revise mode
  decision-guide.md         # heuristics: structure, storage, async, auth/tenancy, hosting, observability, what not to build
  output-format.md          # the document sections, diagrams, decision log, rules
assets/
  architecture.md           # document template → docs/architecture.md
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration. The document always lands at `docs/architecture.md` in the project root so successive versions are diffable in git.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill architecture-design
```
