---
name: design-qa-test-cases
description: >-
  Designs a rigorous, traceable set of QA test cases for a feature by scanning the project's own code — validations, permission checks, state machines, existing tests — instead of generating generic cases from a description alone. Use when asked to "write test cases for this ticket", "what should QA test here", "design the QA plan for this feature", "generate test cases from this Linear/Jira/GitHub issue", "what are the edge cases for X", or "review test coverage before we ship this". Never writes test code — it specifies which manual and automated cases must exist, and at what level, leaving implementation to a developer. The plan is always printed in chat; saving a file or pushing to a QA tool is configured per repo.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Design QA Test Cases Skill

Turns a feature — a ticket, a diff, or a plain description — into a QA test-case specification: happy paths, boundaries, negative paths, permissions, state transitions, data/lifecycle risks, integration/async risks, and cross-cutting UX concerns, each traced back to an acceptance criterion or a concrete signal in the code. This skill **designs** test cases; it never writes test code. It names the case, the level it belongs at (manual, or automated at unit/integration/e2e), and — for an automated one — the framework and a suggested file path. Implementing it is a developer's job.

Installing this skill **in the project's own repo** is the point: a QA researcher on a generic chat can only guess at boundary values, roles, and states — this skill reads the actual validations, policies, and enums, and cross-references the repo's existing tests so QA effort goes only where real coverage is missing.

**Write posture.** Read-only on the project's source — never edits, commits, or pushes code. It has three narrow writes, each opt-in or confirmed: the skill's own config file (only during setup, see below), the test-case report file (only if `output.file` is set), and a push to a configured QA tool (only if configured, and confirmed before each push). The chat report is the deliverable in every case — nothing else depends on those succeeding.

## Setup — Do This First

If `.eagerworks/design-qa-test-cases.json` doesn't exist in the target repo, or the user asks to (re)configure the skill, run the short setup in `references/config.md` before designing anything: language, case format, manual/automated scope, and — after checking what's actually reachable in this harness — an optional push destination. Write the config, say what was written, then continue with the run. Never silently fall back to built-in defaults without offering this — but if the user declines, proceed with defaults for this run and write nothing.

## Intake — What Am I Testing?

Two independent inputs, each resolved from whatever was given — never invent either:

| Given | Do |
| --- | --- |
| A Jira/Linear/GitHub/Notion URL | Fetch it (`gh issue view`, the matching MCP, or ask the user to paste it if nothing can reach it) for the acceptance criteria |
| A branch, diff, or PR reference | Read it for the actual behavior change — the diff is a second, code-grounded source of ACs |
| Plain text describing the feature | Use it as-is; extract ACs from it, and flag anything too vague to test (see `references/techniques.md` → Ambiguities) |
| A Figma link, an attached screenshot, or Figma MCP reachable | Use it for cross-cutting UX cases (layout, states, responsive) |
| No design available | Say so explicitly; mark UI/visual cases ⚪ **Not verifiable without more input** rather than inventing a layout |

Full resolution ladder for both inputs, including what counts as "reachable": `references/intake.md`.

## Scan the Code

Before writing a single case, scan the repo for the signals in the table below — this is what separates a designed case from a guessed one.

| Signal | Yields |
| --- | --- |
| Validations (model/schema, Zod/Yup, DB constraints) | Exact boundary values |
| Policies, guards, middleware, RLS | The role × action matrix actually enforced |
| Enums, state machines, status columns | Legal/illegal transitions |
| Feature flags | Per-flag-state duplication |
| Background jobs, webhooks, external API clients | Async and outage cases |
| **Existing test files** | **Coverage cross-reference — mark cases already covered instead of re-listing them as gaps** |

Full signal table per stack, and how the coverage cross-reference works: `references/code-scan.md`.

## The Eight Lenses

| # | Lens | Yields |
| --- | --- | --- |
| 1 | Happy paths per acceptance criterion | One case per AC |
| 2 | Equivalence partitioning & boundaries | Off-by-one, min/max, empty, unicode, max-length |
| 3 | Negative & error paths | Invalid input, failed dependency, timeout, partial failure |
| 4 | Permissions & roles | One case per role × action the code actually distinguishes |
| 5 | State transitions | Legal and illegal transitions of any state machine touched |
| 6 | Data & lifecycle | CRUD, cascades, soft deletes, uniqueness, concurrency, idempotency |
| 7 | Integration & async | Jobs, webhooks, eventual consistency, retries, third-party outage |
| 8 | Cross-cutting UX | Responsive, a11y, i18n/locale, timezone, empty/loading/error states |

Full checklist, decision rules, and Rails + Node/TypeScript examples for each lens: `references/techniques.md` — the authoritative rubric; never duplicate its detail up here.

## Priority and Coverage Status

Every case gets a priority — **P0** (blocks release), **P1** (should ship with it), **P2** (nice to have) — from how directly it traces to an AC or a data-integrity/security risk, never from padding. Every case also gets a coverage status: `New`, `✅ covered — file:line` (an existing automated test already asserts it), or left inside `### Acceptance criteria with no case` / `### ⚪ Not verifiable without more input` when it can't be specified yet. See `references/output-format.md` for the full report shape.

## Reference Files (read these on demand)

| Task | Read |
| --- | --- |
| Resolving the feature source and the design source, including what "unreachable" means | `references/intake.md` |
| What to extract from the repo per stack, and how the coverage cross-reference works | `references/code-scan.md` |
| The eight lenses in full, traceability, ambiguities, conservatism | `references/techniques.md` |
| Manual vs. automated recommendation, choosing unit/integration/e2e, naming a framework without writing code | `references/automation.md` |
| The markdown report, the three case formats, the `### TEST_CASES` machine block | `references/output-format.md` |
| Pushing the plan to a QA tool, and graceful degradation when it's unavailable | `references/integrations.md` |
| First-run setup and the `.eagerworks/design-qa-test-cases.json` schema | `references/config.md` |

Copyable templates live in `assets/`:

- `assets/design-qa-test-cases.example.json` — starter config
- `assets/test-cases-report.md` — report template, for when `output.file` is set

## Critical Gotchas

1. **Never write test code, ever — not even if asked "now write the Playwright test".** This skill's output is a specification: title, level, framework name, suggested file path. Decline the code and hand back the case; implementing it is a separate task.

2. **Never invent an acceptance criterion, a boundary, or a UI state you weren't given evidence for.** No design available → ⚪, not a guess. An unreachable ticket link → ask for the text, don't fabricate the feature. See `references/techniques.md` → Ambiguities.

3. **A case needs a concrete trigger** — an AC, a validation at `file:line`, a policy, a state enum, or a documented risk area from config. No padding the list to look thorough; a small feature yielding 6 cases is the correct, expected outcome.

4. **Cross-reference existing tests before calling anything a gap.** A case already asserted by `spec/models/coupon_spec.rb:42` is `✅ covered`, not `New` — re-listing covered ground as a gap wastes QA effort on work already done.

5. **No synthetic case IDs.** Don't mint `TC-01`-style identifiers — they aren't stable across re-runs and collide with the real key the destination tool assigns on push. Identify a case by its title; trace it with a `Verifies:` field; report AC coverage as a gap list, not a matrix. See `docs/decision-records/2026-09-07--test-cases-carry-no-synthetic-ids.md`.

6. **Report language is configured, not inferred** — `testCases.language` (default English), never the language the conversation happens to be in. See `references/config.md`.

7. **A push to a QA tool is confirmed, and a missing integration never blocks the chat report.** The plan is delivered in chat regardless of whether `gh`, a Linear/Jira/Notion MCP, or any other destination is reachable. See `references/integrations.md`.

8. **Print the whole report in chat first.** Saving to `output.file` or pushing to a tool is always in addition to the chat report, never instead of it.
