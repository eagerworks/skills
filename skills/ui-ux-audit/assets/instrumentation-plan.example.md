# Tracking plan — <product name>

<!--
Copyable starter the ui-ux-audit skill recommends when dimension 10 finds primary flows
without instrumentation. Copy to docs/tracking-plan.md, replace the example rows with the
project's primary flows, and keep it as the single source of event names.
-->

## Conventions

- **Event names:** `object_action`, snake_case, past tense for completed things (`invoice_sent`), present for intents (`checkout_started`). Defined once in `<lib/analytics.ts | app/services/track.rb>`; call sites never use raw strings.
- **Properties:** ids and enums only — `invoice_id`, `plan: "team"`, `step: "billing"`, `amount_cents`. **Never** emails, names, phone numbers, addresses, card data, or free text.
- **Automatic context** (set once on identify / super-properties): user id or anonymous id, workspace id, platform, app version, locale, current screen.
- **Consent:** no event fires before the user's consent where required (web consent banner wired to the tracker's opt-in; iOS ATT prompt before enabling).
- **Owner:** <name / role> reviews this file when a primary flow changes. Metrics are read in <dashboard link> every <cadence>.

## Primary flows

### Flow: Sign up and onboard

| Event | When | Properties | Metric it feeds | Decision it informs |
|---|---|---|---|---|
| `signup_started` | `/signup` rendered | `source` (invite, organic, campaign) | Funnel top | Which entry points bring users who finish |
| `signup_step_completed` | each step submits successfully | `step` | Step conversion | Which step to simplify first |
| `form_validation_failed` | any field error on the flow | `form`, `field`, `error_code` | Error rate per field | Which label/hint/input type confuses people |
| `signup_completed` | account exists | — | Sign-up conversion | Whether onboarding changes worked |
| `onboarding_step_skipped` | user skips an optional step | `step` | Skip rate | Whether the step should exist at all |
| `<core_action>_completed` (first time) | first project/invoice/post created | `days_since_signup` | **Activation rate** | Whether onboarding leads to value |
| `signup_abandoned` | no completion within 30 min of start (derived) | `last_step` | Drop-off step | Where to focus design effort |

### Flow: <Create the core object>

| Event | When | Properties | Metric it feeds | Decision it informs |
|---|---|---|---|---|
| `<object>_create_started` | form opened | `source` (nav, empty state, shortcut) | Reach per entry point | Where the CTA belongs |
| `form_validation_failed` | field error | `form`, `field`, `error_code` | Error rate per field | Field-level UX fixes |
| `<object>_created` | saved | `<object>_id`, `template` | Completion rate, time to complete | Whether the form is too long |
| `<object>_create_failed` | server error | `error_code` | Failure rate | Reliability vs. UX work |

### Flow: <Checkout / submit>

| Event | When | Properties | Metric it feeds | Decision it informs |
|---|---|---|---|---|
| `checkout_started` | checkout opened | `cart_value_cents`, `currency` | Funnel top | Pricing/placement of the CTA |
| `checkout_step_completed` | each step | `step` | Step conversion | Which step leaks |
| `payment_method_selected` | method chosen | `method` | Method mix | Which methods to keep/add |
| `payment_failed` | gateway/validation error | `error_code` | Payment failure rate | Error copy, retry UX, gateway choice |
| `order_completed` | order persisted (emit server-side) | `order_id`, `amount_cents` | Conversion, revenue per visitor | Everything above |

## Metrics dashboard (what to build first)

1. Step conversion for each primary flow (funnel) — weekly.
2. Activation rate (sign-up → first core action within 7 days) — weekly.
3. Error rate per field on primary forms — weekly, top 10 fields.
4. Time to complete each primary flow (median, p90) — weekly.
5. Retention D1 / D7 / D30 — monthly.

## Tool

Candidate(s) from the audit: <tool 1> (<why it fits the stack>), <tool 2>. **Before choosing, survey the current market** — options, pricing, hosting, and privacy terms change; the candidates are a starting point, not a decision. Record the final choice and the reason here.
