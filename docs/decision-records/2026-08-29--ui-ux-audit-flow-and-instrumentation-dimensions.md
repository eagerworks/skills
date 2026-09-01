# `ui-ux-audit` grades flow simplicity and product instrumentation as dimensions

- **Date:** 2026-08-29

## Context

After the first version of `ui-ux-audit` ([2026-08-29--ui-ux-audit-report-saved-to-docs](2026-08-29--ui-ux-audit-report-saved-to-docs.md)), two requests came in:

1. The audit should be able to say *"this flow is unnecessarily complex, it
   could be simplified like so"* — not only grade the screens, but the length
   of the journey through them.
2. The audit should check and recommend **which flows are worth tracking with
   metrics to make better product decisions, which metrics, and with which
   tool** — while still recommending that the team looks at what the market
   offers today, because the tool landscape changes faster than a skill does.

Both could have been ungraded "advice" sections appended to the report. That
would have broken the skill's core contract (every check has a grade, evidence,
and a `→ Recommendation:`; the Work Plan maps 1:1 to 🔴/🟡) and left the two
most decision-shaping outputs outside the verdict and the plan.

## Decision

1. **Both are graded dimensions** — 9 *Flow simplicity* and 10 *Product
   instrumentation & metrics* — with checks in `references/rubric.md`, rows in
   the scorecard and Work Plan, and the same recommendation rule. They can be
   disabled by config like any other dimension (`dimensions.flows`,
   `dimensions.instrumentation`) and are disclosed when disabled.
2. **Dimension 9 grades only against a flow walk.** A step is "unnecessary"
   only when the audit can name what it collects and show, from model
   validations / API schemas / documented policy, that the outcome doesn't
   require it — or that the system already knows it. Its recommendation is
   always the **proposed step sequence** (which steps merge, move, or
   disappear; what gets prefilled and from where; the files). "Simplify" alone
   is an incomplete finding. Compliance, safety, and irreversible-money steps
   are never 🟡 on taste (`references/flow-simplicity.md`).
3. **Dimension 10 is capped at 🟡** except 10.5 (PII or payment data in event
   payloads is 🔴). Missing analytics never stops a user, so it never makes
   the verdict *Needs work* — but it is a gap, because a team without funnel
   data on its primary flows is deciding on opinion.
4. **Tool recommendations are dated candidates plus a mandatory market-survey
   line.** The audit names one or two tools that fit the stack *as of the
   skill's date* (`references/instrumentation.md` carries the snapshot with its
   date), then always writes: *"Before choosing, survey the current market —
   options, pricing, hosting, and privacy terms change; these candidates are a
   starting point, not a decision."* It never picks for the team, never
   installs, signs up, configures, or adds keys.
5. **A bounded web search is allowed** to keep the candidate list from going
   stale: at most two searches, scoped to the stack, only if the agent has a
   search tool and config doesn't set `marketCheck: false`; the queries are
   listed in the report footer alongside the commands and URLs. Without a
   search tool the plan says the list is a snapshot.
6. The recommendation for 10.1 / 10.2 / 10.6 has a fixed shape — the
   **Instrumentation plan**: per primary flow, events with properties → the
   metric they build → the product decision that metric informs → candidate
   tool. A metric without a decision attached is not allowed in the plan.
   `assets/instrumentation-plan.example.md` is the starter the team copies to
   `docs/tracking-plan.md`.

## Consequences

- The report grows by two sections; a *Solid* project still has an empty Work
  Plan, with **Keep:** lines for 9.x and 10.x naming the practice (single-screen
  flows, direct entry points, the tracker and the tracking plan).
- Dimension 9 findings and dimension 10 findings reinforce each other: the
  metric that would prove a simplification worked (step conversion for that
  flow) is named in the same report as the simplification.
- `references/instrumentation.md` contains a dated tool table and must be
  re-checked when the skill is bumped; the market-survey line exists precisely
  so a stale table never becomes a wrong decision in an audited project.
- The Work Plan orders dimension 10 last: instrumentation is how the team
  will *see* the effect of the other fixes, but it never outranks a defect
  users hit today.

## Related

- [`2026-08-29--ui-ux-audit-report-saved-to-docs.md`](2026-08-29--ui-ux-audit-report-saved-to-docs.md) — the recommendation rule and the one-write contract these dimensions inherit.
- [`skills/ui-ux-audit/references/flow-simplicity.md`](../../skills/ui-ux-audit/references/flow-simplicity.md) — the flow walk and simplification patterns.
- [`skills/ui-ux-audit/references/instrumentation.md`](../../skills/ui-ux-audit/references/instrumentation.md) — event sets, metric catalogue, the dated tool landscape.
- [`skills/ui-ux-audit/references/audit-workflow.md`](../../skills/ui-ux-audit/references/audit-workflow.md) — Phase 2b, the bounded market check.
