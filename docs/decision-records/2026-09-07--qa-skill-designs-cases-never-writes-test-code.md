# `design-qa-test-cases` designs test cases and never writes test code

- **Date:** 2026-09-07

## Context

Every plausible way to build a "generate QA tests" skill blurs, sooner or later, into generating test _code_ — the model that produced this collection's other skills is a code review, an audit, an API design: all analysis with no code in the output. A test-case skill is one prompt away from "just write the Playwright spec" and it would be a genuinely useful thing to do. Two problems make that the wrong scope for this skill specifically:

- **Framework diversity is total, not partial.** `pr-review` supports Rails and Node/TypeScript because that covers this collection's actual target studios. QA automation spans Playwright, Cypress, Selenium, RSpec/Capybara, pytest, Robot Framework, Postman/Newman, and in-house harnesses — there is no small dual-stack set that covers "most" QA teams the way Rails+Node covers most of this collection's engineering users. Writing working code for all of them is a different, much larger skill than this one.
- **The user asked for design, explicitly.** The request that started this skill named the deliverable as "test cases as a QA would think about them" — a specification a human QA reviews, prioritizes, and either executes by hand or hands to a developer to automate. Collapsing that into generated code changes who reviews the output and what "correct" means: a wrong boundary value in a spec is a one-line fix a QA catches in review; a wrong boundary value baked into a passing (because self-consistent) automated test is a false sense of coverage that ships.

Leaving the boundary implicit invites scope creep the first time someone asks "now write it" — and a skill that sometimes writes code and sometimes doesn't is a worse contract than one that never does.

## Decision

1. The skill's output is always a **specification**: a case title, what it verifies, its priority, and — for a case recommended for automation — the level (unit/integration/e2e) and a named framework (`references/automation.md`). It never emits test source code, in any language, at any level, including a "just a skeleton" `describe`/`it` block.
2. The skill's name encodes this: `design-qa-test-cases`, not `create-qa-tests` or `generate-qa-tests` — the verb is deliberately about design, not implementation.
3. `SKILL.md`'s first Critical Gotcha states this explicitly and covers the direct request case ("now write the Playwright test") — decline and hand back the specification instead of complying, the same way `pr-review`'s read-only posture holds even when a user's phrasing implies otherwise.
4. Naming a framework and a suggested file path (`references/automation.md`) is not an exception to this — it's information a developer needs to _start_ writing the test, not part of the test itself.

## Consequences

- A future contributor extending this skill toward code generation should treat that as a **new skill**, not a mode of this one — this collection's precedent (`pr-review`'s optional fix loop for _review findings_, never for QA cases) doesn't transfer here without redoing this analysis, since a wrong fix from `pr-review` is caught by a human reviewing the diff, while a wrong automated test can silently pass.
- The eval suite (`evals/design-qa-test-cases/evals.json`) must include a case that directly asks for generated test code and checks the response declines and redirects to the specification — this is a load-bearing behavior, not an edge case.
- This does not prevent a _separate_ skill or a developer's own follow-up task from consuming this skill's `### TEST_CASES` machine block (`references/output-format.md`) to scaffold code — that's a different tool's job, working from this skill's output.

## Related

- [`skills/design-qa-test-cases/SKILL.md`](../../skills/design-qa-test-cases/SKILL.md) — Critical Gotcha #1.
- [`skills/design-qa-test-cases/references/automation.md`](../../skills/design-qa-test-cases/references/automation.md) — the level/framework recommendation this record permits.
- [2026-09-07--first-run-setup-writes-skill-config.md](2026-09-07--first-run-setup-writes-skill-config.md) — the other novel posture this skill introduces.
