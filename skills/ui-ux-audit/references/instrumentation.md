# UI/UX Audit — Product Instrumentation & Metrics

What dimension 10 grades against and what its recommendations contain: the events a primary flow needs, the metrics they enable, the decisions those metrics inform, the privacy rules, and a **dated** landscape of tools. The audit recommends instrumentation; it never installs it.

## Why the UI audit cares

Dimensions 1–9 say what *looks* wrong from the source. Only instrumentation says what users *actually* do: where they drop, how long the flow takes, which errors they hit, whether a simplification (dimension 9) moved the number. A team without funnel data on its primary flows is making UX decisions on opinion; the audit's job is to say so and hand them the minimal plan to change that.

## Event naming

- One convention, written down once (`docs/tracking-plan.md`, or a `track.ts` / `Analytics` module that owns the names): **`object_action`**, snake_case, past tense for completed things (`invoice_sent`), present for intents (`checkout_started`). Never UI words ("clicked blue button").
- Properties are **ids and enums**, not free text or PII: `invoice_id`, `plan: "team"`, `step: "billing"`, `amount_cents`, `currency`, `error_code`.
- Every event carries the same context automatically (user id or anonymous id, workspace/org id, platform, app version, locale, screen) — set it once in the tracker's identify/super-properties, not per call.
- One place to add an event: a wrapper (`track(name, props)`) so the tool can be swapped without touching call sites.

## Minimal event set by flow type

Instrument the **primary flows first**; each needs a start, its real decision points, a completion, and its failure. Page views alone can't build a funnel.

| Flow type | Start | Steps worth an event | Complete | Fail / drop |
|---|---|---|---|---|
| Sign-up / onboarding | `signup_started` | `signup_step_completed {step}`, `onboarding_step_skipped {step}` | `signup_completed`, `onboarding_completed` | `signup_failed {error_code}`, `signup_abandoned {last_step}` |
| Create the core object (project, invoice, post) | `<object>_create_started {source}` | `<object>_field_error {field}` (validation), `<object>_draft_saved` | `<object>_created {id, template?}` | `<object>_create_failed {error_code}`, `<object>_create_abandoned {fields_filled}` |
| Checkout / payment | `checkout_started {cart_value_cents}` | `checkout_step_completed {step}`, `payment_method_selected {method}` | `order_completed {order_id, amount_cents}` | `payment_failed {error_code}`, `checkout_abandoned {step}` |
| Search / browse | `search_performed {query_length, filters}` | `search_result_clicked {position}`, `filter_applied {filter}` | `search_converted {to}` | `search_no_results` |
| Invite / share | `invite_started {source}` | — | `invite_sent {count}`, `invite_accepted` | `invite_failed` |
| Settings / account | `settings_opened {section}` | — | `settings_saved {section, fields_changed}` | `settings_save_failed` |
| Mobile-specific | `app_opened {cold}`, `screen_viewed {name}` | `push_permission_prompted`, `push_permission_granted` | — | `app_crashed` (via the error tracker) |

Every form on a primary flow also gets `form_validation_failed {form, field, error_code}` — it is the cheapest signal of a confusing field there is.

## Metric catalogue — and the decision each one informs

A metric without a decision attached is a vanity number. The Instrumentation plan names both.

| Metric | Built from | The decision it informs |
|---|---|---|
| **Step conversion** (per step of a flow) | `*_started` → `*_step_completed` → `*_completed` | Which step to simplify or remove first (dimension 9); whether a change to that step worked |
| **Drop-off step** | last event before `*_abandoned` / no completion within N minutes | Where to spend design effort; what to prefill or defer |
| **Time to complete** | timestamp `*_started` → `*_completed` | Whether the flow is too long; effect of merging steps |
| **Error rate per field / form** | `form_validation_failed` grouped by `field` | Which label, hint, or input type is confusing (5.1, 5.3, 5.4) |
| **Activation rate** | share of new accounts that complete the core action within N days | Whether onboarding leads to value; what "set up" really requires |
| **Feature adoption** | share of active users emitting a feature's `*_completed` in a period | Keep, promote, or remove a feature; where to put it in the nav (1.x) |
| **Retention (D1 / D7 / D30)** | users active on day N after `signup_completed` | Whether the product delivers repeat value; when to invest in growth vs. core |
| **Failure rate on primary writes** | `*_failed` / `*_started` | Reliability vs. UX work; whether error states (6.3) are being hit |
| **Reach of each entry point** | `*_started {source}` | Which nav/CTA placements work (1.2, 9.6) |
| **Rage / dead clicks** (session replay tools) | repeated clicks on non-interactive or unresponsive elements | Broken affordances, missing loading states (6.1) |

## Privacy and consent checklist (10.5)

- **No PII in properties** — no emails, names, phone numbers, addresses, card data, free-text fields. Use ids; let the tool's `identify` hold traits if the vendor contract allows it.
- **Consent where required** — EU/UK/Brazil and others require consent for non-essential tracking on the web; iOS requires the ATT prompt before cross-app tracking. Tracking must not fire before consent; a banner that isn't wired to the tracker's opt-in is decoration.
- **Respect signals** — `navigator.doNotTrack` / Global Privacy Control where the team chooses to; document the choice.
- **Retention and access** — write down where data lives (region), for how long, and how a deletion request propagates to the analytics vendor.
- **Server-side for sensitive flows** — payment and auth events are safer emitted from the backend (no client PII, no ad-blocker loss).

## Tool landscape — *snapshot as of 2026-08; verify before deciding*

The audit names **one or two candidates** that fit the stack, then always adds: *"Before choosing, survey the current market — options, pricing, hosting, and privacy terms change; these candidates are a starting point, not a decision."* If a web-search tool is available the audit may run one or two bounded searches (`references/audit-workflow.md` → Phase 2b) to confirm the list is current and cite what it found. Nothing here is installed, signed up for, or configured by the audit.

| Category | What it answers | Candidates (dated) | Fit notes |
|---|---|---|---|
| **Product analytics** (events, funnels, retention, cohorts) | "Where do users drop, who comes back" | PostHog (cloud or self-hosted; also flags, replay), Mixpanel, Amplitude, Heap (auto-capture) | First choice for a product with primary flows to measure; all have web + RN SDKs |
| **Privacy-first web analytics** (page-level, no cookies) | "How much traffic, from where" | Plausible, Fathom, Umami (self-hosted), Matomo | Marketing sites or when consent friction must be zero; usually no funnels by user |
| **Google Analytics 4** | traffic + basic events, ads attribution | GA4 | Free, ubiquitous, but consent burden in the EU and weaker product funnels |
| **Customer data pipeline** (collect once, route to many) | "One SDK, many destinations, warehouse copy" | Segment, RudderStack (open source), Jitsu | When the team already owns a warehouse or expects to swap tools |
| **Warehouse-first / SQL** | "Ask any question later" | events table in Postgres/ClickHouse/BigQuery + Metabase/Lightdash/Looker Studio | Rails apps often start here with `ahoy_matey` + Blazer/Metabase; zero vendor lock-in, more work |
| **Session replay & heatmaps** | "What did they actually do on that screen" | PostHog replay, LogRocket, FullStory, Hotjar, Microsoft Clarity (free) | Pair with funnels; heavy on privacy — mask inputs |
| **Error tracking** (already common) | "What broke, for whom" | Sentry, Bugsnag, Honeybadger, Rollbar | Not analytics, but 10.4 accepts it for failure capture |
| **Feature flags / experiments** | "Did the change move the metric" | PostHog flags, LaunchDarkly, Flipper (Rails), Unleash (open source), GrowthBook (open source) | 10.7; PostHog/GrowthBook tie flags to the same events |

Rule of thumb for the recommendation: a Rails monolith with a small team → `ahoy_matey` events + a SQL dashboard *or* PostHog; a React/Next product with a PM → PostHog or Mixpanel/Amplitude; an Expo app → the same product-analytics SDKs (`posthog-react-native`, `@amplitude/analytics-react-native`, `mixpanel-react-native`) plus the ATT prompt; a marketing site → Plausible/Fathom. Say which and why, then the market-survey line.

## Stack snippets (placeholders — never real keys)

```ruby
# Rails — Ahoy: config/initializers/ahoy.rb + a wrapper the views/controllers call
# ✅ correct
class Track
  def self.event(name, **props) = Ahoy.instance.track(name, props.except(:email, :name))
end
Track.event("invoice_sent", invoice_id: invoice.id, amount_cents: invoice.total_cents)
```

```ts
// Next.js — one wrapper in lib/analytics.ts; the vendor is swappable
// ✅ correct
import posthog from "posthog-js";
posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, { api_host: "https://your-posthog-host" }); // key from env, never inline
export const track = (name: EventName, props?: EventProps) => posthog.capture(name, props);
```

```tsx
// Expo — same wrapper, plus consent/ATT before init
// ✅ correct
import PostHog from "posthog-react-native";
export const analytics = new PostHog("your-project-key", { host: "https://your-posthog-host" });
// call requestTrackingPermissionsAsync() (expo-tracking-transparency) before enabling on iOS
```

```ts
// ❌ wrong — vendor call at the call site, PII, inline key
mixpanel.init("abc123-real-looking-key");
mixpanel.track("Clicked Send", { email: user.email, cardLast4: card.last4 });
```

Starter tracking plan to hand the team: `assets/instrumentation-plan.example.md`.
