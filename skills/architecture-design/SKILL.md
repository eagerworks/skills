---
name: architecture-design
description: >-
  Guides a user through defining the v1 architecture of a new system by asking the questions that matter — problem, users and scale, v1 scope, constraints, quality attributes, data and integrations, delivery — one phase at a time, then proposes a justified architecture and writes it up as docs/architecture.md in the project. Use when asked "design the architecture for X", "help me plan the v1 architecture", "what stack / architecture should I use for this", "write an architecture doc for a new system", "system design for my app", or to revise an existing architecture doc after requirements changed.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Architecture Design Skill

A v1 architecture is the **smallest set of structural decisions** that lets a team ship the first version and change course cheaply afterwards. Most of those decisions depend on facts only the user has — who the users are, what must never fail, how many people will build it, when it has to exist. This skill runs a **guided interview** to collect those facts, synthesizes them into drivers, **proposes an architecture with alternatives and rationale**, and writes the result as a document the team can keep.

The skill has **exactly one write**: the finished document is printed in full in chat **and** saved to `docs/architecture.md` at the project's root (`docs/` is created if missing; the file is overwritten on re-run so the project keeps one current architecture doc). Nothing else is created, edited, committed, or pushed.

## Discovery — Do This First

Never ask what the repo can already answer. Before the first question:

1. **Is there a codebase?** Look at the working directory for manifests (`package.json`, `Gemfile`, `pyproject.toml`, `go.mod`, `docker-compose.yml`, `Dockerfile`, IaC files) and read `README.md`, `AGENTS.md`/`CLAUDE.md`, `docs/`. A greenfield request may still land in a repo that fixes the language, framework, hosting, or CI.
2. **Is there a previous doc?** If `docs/architecture.md` exists, read it in full and run in **revise mode** (`references/design-workflow.md` → "Revising an existing doc"): ask only about what changed, keep the decision log.
3. **What did the user already say?** Extract every fact from the request into the phase checklist (`references/interview.md`) and mark it answered — a user who wrote "a B2B SaaS for 20 dental clinics in Uruguay, Rails, launching in 3 months" has answered half of phases 1, 2, and 4 already.

Then start the interview at the first phase with open questions.

## The Interview

Seven phases, asked **one phase per turn**, at most four questions per turn, each with a recommended default the user can accept with "yes". Adaptive follow-ups, skip rules, and defaults per phase: `references/interview.md` — it is the single source of truth for what to ask.

| # | Phase | What it establishes |
|---|---|---|
| 1 | Problem & goals | What the system is for, what "v1 succeeded" means, what is explicitly a non-goal |
| 2 | Users & scale | Who uses it, how many, from where, the realistic load in the first year |
| 3 | Functional scope | The v1 feature list and what is deliberately deferred |
| 4 | Constraints | Team size and skills, budget, deadline, compliance, systems it must live with |
| 5 | Quality attributes | Availability, latency, data sensitivity, consistency — ranked, not all "high" |
| 6 | Data & integrations | Core entities, volumes, retention, third-party services, inbound/outbound APIs |
| 7 | Delivery & operations | Hosting, environments, CI/CD, observability, who is on call |

Stop asking when the "enough to propose" checklist at the end of `references/interview.md` is satisfied — not before, and not after.

## From Answers to Architecture

1. **Drivers**: turn the answers into 3–6 ranked architectural drivers (the constraints and quality attributes that actually shape the design).
2. **Proposal**: apply `references/decision-guide.md` — default to the simplest structure that satisfies the drivers (usually a modular monolith, one relational database, a job queue, managed hosting) and deviate only when a driver demands it. Every major decision lists the alternatives considered and why they lost.
3. **Confirm**: present the proposal in chat as a short summary (structure, storage, hosting, top 3 decisions) and iterate once with the user before writing the doc.
4. **Deliver**: fill `assets/architecture.md` following `references/output-format.md`, print it in full, then save it.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The questions per phase, follow-ups, skip rules, defaults, and the "enough to propose" checklist | `references/interview.md` |
| Running the whole thing end-to-end: discovery, interview, synthesis, proposal, delivery, revise mode | `references/design-workflow.md` |
| Choosing structure, storage, async, auth, tenancy, hosting, observability — and what not to build in v1 | `references/decision-guide.md` |
| The exact document sections, diagrams, decision-log table, and rules | `references/output-format.md` |

Copyable assets live in `assets/`:
- `assets/architecture.md` — the document template; fill it in and save it as `docs/architecture.md`

## Critical Gotchas

1. **One phase per turn, at most four questions.** A wall of twenty questions gets shallow answers or none. Ask, wait, adapt the next phase to what came back.

2. **Every question carries a default.** "How many users in year one? (If unsure, I'll assume < 1,000 and design for 10×.)" A user who can say "yes" to all defaults finishes the interview in seven turns.

3. **Never ask what the code already answers.** If `Gemfile` exists, the language is decided; if `.github/workflows/deploy.yml` deploys to Fly, so is the host. Confirm in one line, don't ask.

4. **Boring by default, and say why.** A modular monolith on one relational database with a queue is the v1 answer until a driver says otherwise. Microservices, event sourcing, a second database, or Kubernetes need a named driver in the doc — never "for scalability" alone.

5. **Every major decision gets alternatives and a rationale.** A decision-log row with an empty "alternatives" column is not a decision, it's a habit.

6. **Push back once, then respect the call.** If the user wants something the drivers don't justify, state the trade-off in two sentences. If they reaffirm it, record it as their decision with their reason — don't relitigate, don't silently design around it.

7. **Record open questions instead of inventing answers.** Unknown load, unconfirmed compliance scope, an integration with no docs yet — these go in "Risks & open questions" with an owner, not in the design as invented numbers.

8. **One write, nothing else.** The only file you create is `docs/architecture.md`. No scaffolding, no `mkdir` of the proposed structure, no config files, no `git commit`, `git push`, or PR. The doc *describes* the system — it doesn't start building it.

9. **Print the whole document in chat, then save the identical text.** The user asked for both; a chat summary that says "see the file" is not the deliverable.
