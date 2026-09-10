# Design QA Test Cases — Manual vs. Automated

Every case in the plan gets a recommendation — `Manual`, `Automated`, or `Both` — with the reason stated, and, for an automated one, a level and a framework. **This skill never writes the test code**, at any level — see `SKILL.md` gotcha #1. It names what should exist; a developer (or a separate implementation task) writes it.

## Recommendation Rules

- **Automate** when the check is deterministic, cheap to assert, repeatable every release, and sits on a P0/P1 path. Most of Lenses 2–7 (boundaries, negative paths, permissions, state transitions, data/lifecycle, integration) are automation candidates by nature — they're exact input/output pairs.
- **Manual** when it needs human judgment the assertion can't cheaply capture: visual fidelity against a design, copy/tone review, an exploratory pass around a new flow, or a one-off migration/release verification that won't repeat. Most of Lens 8 (cross-cutting UX) leans manual unless the repo already has visual-regression or a11y-audit tooling in place — check for that before defaulting to manual.
- **Both** when a smoke version of the case is worth automating (does the flow complete at all) while a broader exploratory pass around it stays manual (does it _feel_ right).

Never default a case to `Manual` just because it's easier to write down — if it's a deterministic assertion, say `Automated` even if the repo doesn't have a matching test yet. The `New` vs. `✅ covered` status (from `references/code-scan.md`) is orthogonal to this recommendation: an automated case can be `New` (nobody's written it) or already `✅ covered`.

## Level Ladder

Pick the cheapest level that can actually catch the failure — don't reach for e2e by default:

| Level | Use when | Example |
| --- | --- | --- |
| `unit` | The behavior lives in one function/method/class with no I/O | A boundary on a validation, a pure calculation, a policy method in isolation |
| `integration` | The behavior spans a few collaborators — a service touching the DB, a route handler and its middleware | A negative path through a real DB constraint, a permission check through actual middleware |
| `e2e` | The behavior can only be observed through the real UI/API surface end to end | The full checkout flow with a coupon applied, a cross-page redirect after an action |

```markdown
✅ correct
Case — Coupon at exactly the minimum cart total (boundary)
Automate: Yes · Level: unit · Framework: RSpec · Suggested: spec/models/coupon_spec.rb

❌ wrong
Case — Coupon at exactly the minimum cart total (boundary)
Automate: Yes · Level: e2e · Framework: Playwright
# a pure validation boundary doesn't need a browser to catch a regression
```

## Naming the Framework Without Writing the Code

1. Detect from the repo: `package.json` devDependencies (`@playwright/test`, `cypress`, `vitest`, `jest`), `Gemfile` (`rspec-rails`, `capybara`, `cucumber-rails`), `pyproject.toml` (`pytest`).
2. If more than one framework exists at different levels (e.g. Vitest for unit, Playwright for e2e), name the one matching the case's level.
3. If none is detected, or the repo has no test setup at all, say so and name the level only — don't invent a framework the project doesn't use. `testCases.automation.framework` in config can override auto-detection when a repo's convention isn't obvious from manifests alone.
4. The "suggested file path" follows the repo's own existing convention (colocated `*.test.ts` next to the source, or a mirrored `spec/` tree) — inferred from where similar existing tests already live, never a path the skill invents from scratch.

## Scope Filtering

`testCases.scope` in config (`["manual"]`, `["automated"]`, or both — the default) filters what's _printed_, not what's _derived_ — the skill still reasons about the full set of cases internally (so coverage cross-referencing and traceability stay complete) and then shows only the requested scope. When `scope` excludes one side entirely, say so once at the top of the report rather than silently omitting a column.
