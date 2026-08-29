# UI/UX Audit — Flow Simplicity

What dimension 9 grades against, how to build the evidence, and the patterns a recommendation should name. The rubric row for each check is in `references/rubric.md`; this file is the method.

## The question

A flow is as simple as it can be when **every screen collects something the outcome requires, nothing already known is asked again, and each screen asks for at most one real decision.** Anything beyond that is cost the user pays without getting anything back — and it is usually where funnels leak (dimension 10 is how you'd prove it).

## The flow walk (mandatory evidence)

For each primary flow, walk the screens in order and fill this table. It is the only admissible evidence for a 🟡/🔴 on 9.x — a flow "feels long" is not a finding.

| # | Step / screen | File(s) | Data collected | Required for the outcome? (where it says so) | Already known? (from where) | Decisions asked |
|---|---|---|---|---|---|---|
| 1 | `/signup` | `app/views/registrations/new.html.erb` | email, password | yes — `User` validates presence | no | 0 |
| 2 | `/onboarding/company` | `app/views/onboarding/company.html.erb` | name, size, industry, VAT id, address ×4, phone | **no** — `Company` requires only `name` | `name` — in the invite token | 1 (size) |
| 3 | `/onboarding/plan` | `app/views/onboarding/plan.html.erb` | plan | **no** — free tier exists; paywall at export | no | 1 (plan) + 1 (billing cycle) |
| 4 | `/onboarding/invite` | `app/views/onboarding/invite.html.erb` | teammate emails | no — skippable, but "Skip" is a text link below the fold | no | 1 |
| ✓ | `/dashboard` | | | outcome reached after 4 screens, ~14 fields, 4 decisions | | |

"Required for the outcome" is decided by the code, not by the audit: model validations, API request schemas, `required` attributes that the backend actually enforces, or a documented policy (README / product doc / compliance note). If you can't find where a field is required, it is *not required* for grading purposes — say so in the evidence.

Sources for the walk: the route file, the step components/views, the models or schemas each step submits to, the invite/session/locale data available at each step, and — with a runtime pass — screenshots of each step at the primary viewport.

## Simplification patterns (name them in the recommendation)

| Pattern | Use when | What the recommendation says |
|---|---|---|
| **Merge steps** | Two consecutive steps collect ≤ 5 fields combined and no step depends on the previous one's result | "Merge `/onboarding/company` into `/signup` as one form; keep `name` only" |
| **Defer to first need** | A step collects data only a *later* feature needs (billing, address, integrations) | "Ask for the plan at the first paywalled action (`ExportsController#new`), not during sign-up" |
| **Prefill / infer** | The system already holds the value (invite token, session, previous record, locale, IP country) | "Prefill `company_name` from `Invite#company` and hide the field; prefill `country` from `I18n.locale`" |
| **Default + progressive disclosure** | A screen asks several co-equal choices where one option covers most users | "Default to the *Team* plan monthly; move billing cycle and add-ons behind 'More options'" |
| **Undo instead of confirm** | A confirmation guards a reversible action | "Remove `data-turbo-confirm` on profile save; flash 'Saved · Undo' for 10 s" |
| **Single screen for short flows** | The whole flow is ≤ 5 fields and 1 decision | "Replace the 3-step `Stepper` in `NewProjectWizard.tsx` with one form; keep the stepper for imports only" |
| **Save and resume** | A flow is long *for a reason* (compliance, data entry) | "Persist `onboarding_step` on `User` and route `/onboarding` to the last incomplete step" |
| **Direct entry point** | The most common task starts ≥ 3 navigations deep | "Add '+ New invoice' to the primary nav and to the empty state of `/invoices`; return to the list after save with the new row highlighted" |
| **Skip that is a real button** | An optional step's skip is hidden or styled as a secondary link | "Make 'Skip for now' a visible secondary button beside the primary CTA on `/onboarding/invite`" |
| **Collapse the review screen** | A "Review" step just repeats the form with no edit affordance | "Drop `/checkout/review`; show an inline summary on `/checkout/confirm` with 'Edit' links per section" |

## When *not* to simplify

Grade these 🟢 (or ⚪ with the question) rather than 🟡:

- **Compliance and safety** — KYC, age/consent capture, 2FA enrolment, terms acceptance, data-residency choices. Recommend *placement* (defer until needed, explain why it's asked) but never removal.
- **Irreversible money or data moves** — transfers, deletions, bulk actions, sending to third parties. A confirmation step is correct here (6.5); the recommendation is to make it *good* (name the object, verb on the button), not to remove it.
- **Expert tools** — dense screens for power users who use them daily (back-office, trading, ops consoles). Density is a feature; grade decision load against the user's expertise stated in the docs, and ask the PM if not stated.
- **Steps that branch** — a choice that genuinely changes the next screens (personal vs. business account) is a real decision; recommend a default only if the docs say one path dominates.

## Worked example — the recommendation text the report should produce

Given the walk above:

```markdown
- 🟡 **9.1 Sign-up takes 4 screens for an outcome that needs 1** — `/signup` → `/onboarding/company` → `/onboarding/plan` → `/onboarding/invite` → `/dashboard`; only step 1 collects required data (`User` validations). Evidence: flow walk in § 9, `app/models/company.rb:4` (`validates :name, presence: true` — nothing else), `app/models/invite.rb:12` (token carries `company_id`).
  → Recommendation: Collapse to 2 screens. (1) `/signup`: email, password, and `company_name` prefilled from `Invite#company` when a token is present (hide the field). (2) After first login, an optional "Set up your workspace" panel on `/dashboard` for size/industry/VAT/address — none required by `Company`. Defer plan selection to `ExportsController#new` (the first paywalled action) with *Team · monthly* as the default; move "Invite teammates" to the dashboard empty state with a visible "Skip for now" button.
- 🟡 **9.5 Confirmation on a reversible save** — `app/views/companies/_form.html.erb:31` uses `data-turbo-confirm="Are you sure?"` on the profile update.
  → Recommendation: Remove the confirm; on update, flash "Company profile saved · Undo" that restores the previous attributes (`Company#previous_changes`) for 10 s.
```

The Flow map row for the same flow (`references/output-format.md` § 4):

```markdown
| Sign up and onboard | 4 · 14 fields · 4 decisions | `/signup`, `/onboarding/{company,plan,invite}` | 2 · 3 fields · 0 decisions (plan deferred, company prefilled) | −2 steps · −11 fields |
```
