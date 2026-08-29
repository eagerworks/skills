# UI/UX Audit — Rubric

This file is authoritative. Grade **every** check below (unless its dimension is disabled by config), record evidence, write a recommendation for every check, and roll each dimension up to its worst check.

Grades: 🔴 **Blocker** · 🟡 **Gap** · 🟢 **Solid** · ⚪ **Unverifiable from code**. Definitions, the verdict rule, and the recommendation rule are at the end.

Ten dimensions: 1–8 grade the interface as built; 9 grades whether each primary flow is longer than its outcome requires; 10 grades whether the team can measure those flows and decide with data.

"Primary flow" means one of the journeys identified in Discovery step 3. A defect on a primary-flow screen is graded one level harsher than the same defect elsewhere (🟡 → 🔴) when it stops or misleads the user.

## Dimension 1 — Information architecture & navigation

*Can a first-time user find the thing they came for, and get back?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 1.1 Route map is coherent | Routes group by resource/task; URL names match nav labels; no duplicate paths to the same screen | 🟡 orphan routes, inconsistent naming (`/settings` vs `/account/prefs`) |
| 1.2 Persistent primary navigation | A shared layout renders the same nav on every authenticated screen with the current section marked (`aria-current="page"` / active style) | 🔴 primary screens with no way to reach the rest of the app; 🟡 no active-state marker |
| 1.3 Back and deep links work | Every screen is addressable by URL (or a typed route in RN); browser back / hardware back returns to the previous screen; modals/drawers don't trap history | 🔴 state-only screens (a wizard step only reachable via in-memory state) on a primary flow; 🟡 elsewhere |
| 1.4 No dead ends | Every non-final screen has a next action; final screens (success, empty results) link onward | 🟡 a success/empty screen with no CTA |
| 1.5 Error screens exist | Custom 404 / 403 / 500 (or RN error boundary) with nav back; unauthenticated redirect preserves the intended destination | 🟡 framework default error pages; 🔴 a white screen on a caught error in a primary flow |
| 1.6 Hierarchy is visible | One `h1` per screen, page titles set (`<title>`, `document.title`, `Stack.Screen options.title`), breadcrumbs where depth ≥ 3 | 🟡 missing titles or multiple `h1` |
| 1.7 Destructive vs. primary actions are distinguishable in placement | Primary CTA is consistently placed (same corner/end-of-form) and visually dominant; destructive actions are separated | 🟡 primary and destructive actions styled or positioned alike |

Evidence: `grep -rn "aria-current\|active" app/views/layouts`, the route file, `rg "notFound\|not-found\|ErrorBoundary" src app`, screenshots from the browser pass.

## Dimension 2 — Visual consistency & design system

*Is there one visual language, or one per developer?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 2.1 Tokens exist and are the source | Colors, spacing, radii, type scale defined once (`tailwind.config.*` theme, CSS custom properties, `theme.ts`, SCSS variables) | 🟡 no token file — values are literal everywhere |
| 2.2 Tokens are actually used | Hardcoded hex/rgb/px values in views/components are rare (< 5 % of color usages, and each justified) | 🟡 widespread hardcoded values; 🔴 the same semantic color (e.g. "danger") has ≥ 3 different hex values in use |
| 2.3 Shared components for shared patterns | Buttons, inputs, modals, tables, cards, toasts come from one place (component lib or `components/ui`) | 🟡 ≥ 2 competing implementations of the same primitive |
| 2.4 One icon set | A single icon library / sprite; consistent size and stroke | 🟡 mixed libraries or inline SVGs with differing sizes |
| 2.5 Typography scale | ≤ 6 font sizes in use, mapped to roles (display, h1–h3, body, caption); ≤ 2 font families | 🟡 > 8 sizes or arbitrary `text-[13px]`-style escapes |
| 2.6 Theme parity | If dark mode / brand themes exist, every token has a value in each theme and no component hardcodes light-only colors | 🟡 partial theme; 🔴 unreadable text in a shipped theme on a primary flow |
| 2.7 Spacing rhythm | Spacing values come from the scale (4/8 px grid or the token set) | 🟡 arbitrary values (`margin: 13px`, `p-[7px]`) common |

```bash
# Evidence
grep -rnoE "#[0-9a-fA-F]{3,8}\b" app/views app/components src/components 2>/dev/null | wc -l
grep -rnoE "#[0-9a-fA-F]{6}\b" app/views src 2>/dev/null | cut -d: -f3 | sort | uniq -c | sort -rn | head
rg -o "text-\[[0-9]+px\]|p-\[[0-9]+px\]|m-\[[0-9]+px\]" src app 2>/dev/null | wc -l
rg -l "styled\.|css\`|\.module\.css|className=" src | head   # competing styling approaches
```

```tsx
// ✅ correct — semantic token, shared primitive
<Button variant="destructive">Delete project</Button>

// ❌ wrong — one-off styled element with literal color
<button style={{ background: "#e11d48", padding: 7 }}>Delete project</button>
```

## Dimension 3 — Layout & responsiveness

*Does it work on the screens people actually use?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 3.1 Primary viewports handled | Layout adapts at least at mobile (≤ 414 px), tablet, desktop; RN: small phones (iPhone SE / 360 px Android) and large | 🔴 primary flow unusable (overlap, cut-off CTA) at a primary viewport; 🟡 awkward but usable |
| 3.2 No horizontal overflow | No page-level horizontal scroll at any primary viewport; wide tables/code scroll inside their own container | 🟡 page scrolls sideways |
| 3.3 Touch targets | Interactive elements ≥ 44×44 px (web: 24×24 minimum per WCAG 2.5.8, 44 recommended); ≥ 8 px between adjacent targets | 🔴 < 24 px targets on a primary flow; 🟡 24–44 px or crowded |
| 3.4 Safe areas and insets | RN: `SafeAreaView` / `useSafeAreaInsets` on every screen root; web PWA: `env(safe-area-inset-*)` where fixed bars exist | 🟡 content under notch/home indicator |
| 3.5 Text reflows and zooms | Web: usable at 200 % zoom / 320 px CSS width; no `maximum-scale=1` / `user-scalable=no`; RN: respects font scaling or documents `allowFontScaling` exceptions | 🔴 zoom disabled; 🟡 text truncation/overlap at 200 % |
| 3.6 Fixed elements don't block content | Sticky headers/footers/FABs leave the primary content and CTA reachable at small heights (landscape phone, 568 px) | 🟡 CTA hidden behind a fixed bar |
| 3.7 Breakpoints are consistent | One breakpoint set (framework defaults or tokens), not per-component magic numbers | 🟡 ad-hoc `@media (max-width: 743px)` values |

Evidence: `grep -rn "user-scalable\|maximum-scale" app/views/layouts src/app`, `rg "SafeAreaView|useSafeAreaInsets" src app`, `rg "@media" --stats`, screenshots at 375 / 768 / 1280 px from the browser pass. Without a runtime, 3.1, 3.2, and 3.6 are ⚪ unless the source makes the failure certain (e.g. a `width: 1200px` wrapper with no media query).

## Dimension 4 — Accessibility (WCAG 2.2 AA)

*Can someone using a keyboard, a screen reader, or low vision complete the primary flows?* Full checklist and thresholds: `references/accessibility.md`.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 4.1 Semantic structure | Landmarks (`header/nav/main/footer` or roles), heading order without skips, lists as lists, buttons as `<button>`, links as `<a href>` | 🔴 clickable `div`/`span` as the only way to trigger a primary action; 🟡 heading skips, missing `main` |
| 4.2 Images and icons | Meaningful images have descriptive `alt` / `accessibilityLabel`; decorative ones `alt=""` / `aria-hidden`; icon-only buttons have an accessible name | 🔴 icon-only primary action with no name; 🟡 missing `alt` on content images |
| 4.3 Form labels | Every input has a programmatic label (`<label for>`, `aria-label`, `aria-labelledby`, RN `accessibilityLabel`); placeholders are not labels | 🔴 unlabeled inputs on a primary flow; 🟡 elsewhere |
| 4.4 Keyboard reachability & order | All interactive elements reachable by Tab in a logical order; no positive `tabindex`; modals trap focus and restore it on close; Escape closes overlays | 🔴 keyboard-unreachable primary action; 🟡 focus not restored, no Escape |
| 4.5 Visible focus | Focus indicator is visible on every interactive element (no global `outline: none` without a replacement) | 🔴 `outline: none` / `focus:outline-none` globally with no replacement; ⚪ if it can't be seen without runtime |
| 4.6 Color contrast | Text ≥ 4.5:1 (≥ 3:1 for large text), UI component boundaries and focus rings ≥ 3:1 — computed from tokens or measured in the browser | 🔴 primary text/CTA below 3:1; 🟡 below 4.5:1; ⚪ if not computable |
| 4.7 Motion, language, and live regions | `prefers-reduced-motion` respected for non-essential animation; `<html lang>` set; dynamic status messages use `aria-live`/`role="status"` or RN `AccessibilityInfo.announceForAccessibility` | 🟡 any missing |

```bash
# Evidence
rg -n "<img(?![^>]*alt=)" app/views src --pcre2 | wc -l          # images with no alt at all
rg -n "onClick=" src --glob '*.tsx' | rg -v "<button|<a |<Link|role=" | head
rg -n "outline:\s*none|outline-none|focus:outline-none" app/assets src | head
rg -n "tabindex=\"[1-9]|tabIndex=\{[1-9]" app/views src | head
grep -rn "<html" app/views/layouts src/app/layout.tsx | grep -v "lang="
rg -n "accessibilityLabel|accessibilityRole" src | wc -l        # RN coverage
```

```erb
<%# ✅ correct %>
<button type="button" aria-label="Close dialog"><%= icon("x") %></button>

<%# ❌ wrong %>
<div class="close" onclick="closeModal()"><%= icon("x") %></div>
```

## Dimension 5 — Forms & input

*Can the user get the data in right the first time, and fix it when they don't?* State expectations: `references/ui-states.md`.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 5.1 Labels and hints | Visible label above/beside every field; helper text for format expectations; required/optional marked consistently (one convention, not both) | 🟡 placeholder-only labels or mixed required conventions |
| 5.2 Validation placement and timing | Errors render next to the field they belong to, are associated (`aria-describedby`, `aria-invalid`), and appear on blur/submit — not on every keystroke before the user finishes | 🔴 errors only in a flash/toast with no field mapping on a primary form; 🟡 keystroke-nagging or unassociated |
| 5.3 Error wording | Says what's wrong and how to fix it ("Enter a date after today"), not codes or "Invalid input" | 🟡 generic or technical messages |
| 5.4 Input types and autofill | `type=email/tel/url/number`, `inputmode`, `autocomplete` (`email`, `new-password`, `one-time-code`, `postal-code`); RN `keyboardType`, `textContentType`, `autoComplete` | 🟡 all fields `type=text`; 🔴 password/OTP fields blocking autofill on a primary flow |
| 5.5 Submit state | Submit disables or shows progress during the request; double-submit prevented; success confirmed | 🟡 no pending state; 🔴 double-submit creates duplicates (e.g. payments) |
| 5.6 Data preserved on error | Server-side validation failure re-renders with the user's input intact; multi-step forms keep prior steps | 🔴 primary form wipes input on error; 🟡 elsewhere |
| 5.7 Sensible defaults and grouping | Related fields grouped (`fieldset`/`legend` or a section heading); defaults prefilled where known (country, currency); long forms chunked | 🟡 one flat 20-field form |

```bash
# Evidence
rg -n "type=\"text\"|type=\"password\"" app/views src | rg -v "autocomplete|autoComplete" | head
rg -n "aria-describedby|aria-invalid|errors\[" app/views src | wc -l
rg -n "disabled=.*(loading|pending|submitting)|aria-busy" src app/views | wc -l
```

## Dimension 6 — Feedback & system states

*Does the UI tell the user what's happening, what happened, and what to do next?* The canonical matrix is `references/ui-states.md`; recommend `assets/ui-states.example.md` when a screen has none of it.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 6.1 Loading state | Async screens/regions show a skeleton or spinner within 100 ms and don't shift layout when data lands; RN uses `ActivityIndicator`/skeletons | 🟡 blank area while loading; 🔴 a primary-flow screen that appears broken until data arrives |
| 6.2 Empty state | Lists/dashboards with no data explain why and offer the first action ("No projects yet — Create one") | 🟡 an empty table header or blank region |
| 6.3 Error state | Failed fetches/mutations show an inline message with a retry; error boundaries wrap route segments | 🔴 unhandled rejection → white screen or silent failure on a primary flow; 🟡 console-only errors |
| 6.4 Success feedback | Mutations confirm (toast/inline/redirect with flash); confirmation is announced (`role="status"`) and dismissible | 🟡 silent success |
| 6.5 Destructive confirmation and undo | Irreversible actions confirm with the object named ("Delete *Q3 budget*?") or offer undo; confirm button labelled with the verb, not "OK" | 🔴 one-click irreversible delete on a primary object; 🟡 generic `confirm("Are you sure?")` |
| 6.6 Offline / degraded | Web: fetch failures distinguish network from server errors; RN: `NetInfo` or equivalent surfaces offline and queues or blocks writes | 🟡 no distinction; 🔴 RN app with no offline handling on a primary write |
| 6.7 Permission-denied and expired-session | 403 and session expiry show a clear screen or redirect with a message, not a generic error | 🟡 generic error |

```bash
# Evidence
rg -n "Skeleton|isLoading|loading\?|ActivityIndicator|Suspense" src | wc -l
rg -n "No .* yet|empty-state|EmptyState|\.empty\?" app/views src | wc -l
rg -n "ErrorBoundary|error\.tsx|rescue_from" src app | head
rg -n "window\.confirm|confirm\(|data-turbo-confirm|Alert\.alert" app/views src | head
rg -n "NetInfo|navigator\.onLine" src | wc -l
```

## Dimension 7 — Content & microcopy

*Does the text sound like one product, and can it ship in another language?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 7.1 Buttons say what they do | CTAs are verb + object ("Save changes", "Send invite"), not "Submit"/"OK"/"Yes" | 🟡 generic labels on primary actions |
| 7.2 Consistent terminology | One name per concept across screens (not "workspace" here and "team" there); consistent casing (sentence vs. Title) | 🟡 synonyms in the UI for the same object |
| 7.3 Plain language | No internal jargon, IDs, or stack traces shown to users; error copy in user terms | 🟡 jargon; 🔴 raw exception text rendered on a primary flow |
| 7.4 Strings are externalized | User-facing text goes through i18n (`t()`, `I18n.t`, `useTranslation`, `FormattedMessage`) if the app has locales, or is at least centralized | 🟡 hardcoded strings in an app with i18n; ⚪/🟢 if single-locale by stated policy |
| 7.5 Pluralization, dates, numbers | Plural rules via the i18n lib (not `count + " items"`), dates via `Intl`/`l()`, currency via `number_to_currency`/`Intl.NumberFormat` | 🟡 string-concatenated plurals or hand-formatted dates |
| 7.6 Truncation and length | Long user content (names, titles) truncates with ellipsis or wraps deliberately; layouts tested with long strings | 🟡 overflow with long strings (⚪ without runtime unless CSS makes it certain) |
| 7.7 Helpful empty/error copy | Empty and error states say *why* and *what next* (ties to 6.2/6.3) | 🟡 "Something went wrong." with no next step |

```bash
# Evidence
rg -n ">[A-Z][a-z].*<" app/views --glob '*.erb' | rg -v "t\(|I18n" | wc -l    # literal strings in ERB (rough)
rg -n "\+ ?['\"] ?(item|user|file)s?['\"]" src app/views | head                  # concatenated plurals
rg -n "toLocaleDateString|Intl\.|strftime|\bl\(" src app | wc -l
rg -n ">(Submit|OK|Yes|No)<" app/views src | head
```

## Dimension 8 — Performance as experienced

*Does the UI feel fast and stable, independent of backend speed?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 8.1 Images are sized and optimized | `width`/`height` or aspect-ratio on `<img>`; `next/image`/`image_tag` with size; modern formats; lazy loading below the fold | 🟡 unsized images (layout shift); 🔴 multi-MB hero images on a primary screen |
| 8.2 Fonts don't block or shift | `font-display: swap`/`optional`, preloaded primary font, ≤ 2 families / ≤ 4 weights; RN fonts loaded before first render (`useFonts` + splash) | 🟡 FOIT/FOUT sources; unbounded weights |
| 8.3 Long lists are bounded | Pagination, infinite scroll, or virtualization (`FlatList`/`FlashList`, `react-window`) for lists that can exceed ~100 rows | 🔴 RN `ScrollView` + `.map` over an unbounded list on a primary screen; 🟡 web unpaginated tables |
| 8.4 Perceived load on primary screens | Above-the-fold content renders without waiting on non-critical data (streaming/Suspense, Turbo frames, skeletons) | 🟡 whole screen blocked on the slowest query |
| 8.5 No jank sources | Animations use `transform`/`opacity` (or RN `useNativeDriver`/Reanimated); no layout-thrashing scroll handlers; heavy work off the main thread | 🟡 `top/left/height` animations, unthrottled scroll listeners |
| 8.6 Asset budget sanity | Web: no obviously unused large deps (`moment` + `date-fns`, three chart libs); build output or bundle analyzer exists; RN: Hermes enabled | 🟡 duplicate heavy libraries; ⚪ if no build output to inspect |
| 8.7 Measured | Lighthouse CI / Web Vitals / Sentry performance or an equivalent exists | 🟡 nothing measures it; ⚪ if it lives in a dashboard you can't see |

```bash
# Evidence
rg -n "<img" app/views src | rg -v "width=|height=|next/image|image_tag" | wc -l
rg -n "font-display|preload.*font|useFonts" app src | head
rg -n "ScrollView" src | rg -B2 -A6 "\.map\(" | head
jq '.dependencies | keys' package.json 2>/dev/null | rg "moment|dayjs|date-fns|lodash|chart|d3"
find public/images app/assets/images src/assets -size +500k 2>/dev/null
```

## Dimension 9 — Flow simplicity

*Does each primary flow take the minimum steps, decisions, and data the outcome actually requires?* Method, simplification patterns, and a worked example: `references/flow-simplicity.md`.

Evidence for this dimension is a **flow walk**: for every primary flow, the screens in order (`route → file`), the data each step collects, whether that data is required for the outcome, whether the system already knows it, and how many decisions the screen asks for. Without the walk, no 9.x check may be graded 🟡/🔴.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 9.1 Step count matches the outcome | Each primary flow reaches its outcome in the fewest screens its required data allows; every step collects something the outcome needs | 🟡 a step that collects nothing required, or two steps whose fields fit on one screen; 🔴 a primary flow with a mandatory step users can't satisfy (data they don't have yet) |
| 9.2 Value before non-essential data | The user reaches the first useful screen before being asked for optional/profile/billing data ("ask when needed") | 🟡 an up-front wall (full profile, company details, payment) before first use that no stated policy requires |
| 9.3 Nothing already known is asked again | Fields the system can prefill or infer (org from invite token, email from session, country from locale, address from previous order) are prefilled or omitted | 🟡 repeated or inferable fields; 🔴 user must re-enter a whole step after a failure on a primary flow (ties to 5.6) |
| 9.4 Decision load per screen | ≤ 1 primary decision per screen; choices have a sensible default; advanced options collapsed behind progressive disclosure | 🟡 screens with several co-equal choices and no default (pick a plan *and* a region *and* a template on one screen) |
| 9.5 Confirmations only where irreversible | Confirmation dialogs guard only irreversible actions (6.5); reversible ones use undo; no modal-in-modal or wizard-in-modal | 🟡 "Are you sure?" on save/edit/reversible actions, nested modals, a wizard opened inside a dialog |
| 9.6 A direct happy path exists | The most frequent task has a direct entry (nav item, "+ New", quick action, keyboard shortcut) and returns the user where they were | 🟡 a common task needs ≥ 3 navigations to start or dumps the user on an unrelated screen after |
| 9.7 Exit and resume | Multi-step flows longer than ~5 fields can be left and resumed (draft, saved progress, URL per step) — or are short enough not to need it | 🟡 long flows that lose all progress on back/refresh/exit |

```bash
# Evidence — build the flow walk from these, then read each step's view/component
bin/rails routes 2>/dev/null | rg -i "sign|onboard|checkout|new|wizard|step"
rg -n "step|wizard|Stepper|multi-step|currentStep" src app/views app/javascript | head
rg -n "validates .*presence|required:|required\b" app/models app/views src | head -40   # what each step demands
rg -n "confirm\(|data-turbo-confirm|Alert\.alert" app/views src | rg -iv "delete|destroy|remove|cancel" | head  # confirms on reversible actions
rg -n "invite|token|prefill|defaultValue|value: current_" app src | head             # what could be prefilled
```

```text
❌ wrong recommendation:  "Simplify the sign-up flow."
✅ correct recommendation: "Collapse sign-up from 4 screens to 2: (1) email + password + company name
   prefilled from the invite token (app/views/registrations/new.html.erb); (2) a single optional
   'Set up your workspace' screen after first login for the 9 company-profile fields, none of which
   app/models/company.rb requires. Move plan selection to the first paywalled action; drop the
   data-turbo-confirm on the profile save (reversible) and add an 'Undo' flash instead."
```

A step is "unnecessary" only when the audit can name what it collects **and** show that the outcome doesn't require it (check the model validations / API contract) or that the system already has it. Legal, compliance, and safety steps (KYC, consent, 2FA enrolment, irreversible money moves) are 🟢 or ⚪ with the question for the PM — never 🟡 on taste.

## Dimension 10 — Product instrumentation & metrics

*Can the team see whether the primary flows work, and make product decisions with data instead of opinions?* Event sets, metric catalogue, privacy checklist, and the (dated) tool landscape: `references/instrumentation.md`.

**Grade cap:** absence of analytics never stops a user, so this dimension yields at most 🟡 — except 10.5, where PII in event payloads is 🔴.

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 10.1 A product-events layer exists | A tracker is a dependency **and** initialized in the app shell (PostHog, Mixpanel, Amplitude, Segment, GA4, Plausible, Ahoy in Rails, an in-house events table, …) | 🟡 nothing, or a dependency that's never initialized |
| 10.2 Primary flows are instrumented | Every primary flow emits a start, its key steps, and a completion event, so a funnel can be built | 🟡 page views only, or some flows only |
| 10.3 Events are named and shaped consistently | One convention (`object_action`, snake_case, e.g. `invoice_sent`), defined centrally (constants, `track.ts`, a tracking plan doc), properties typed | 🟡 ad-hoc string literals at call sites, mixed conventions |
| 10.4 Failures and drop-offs are captured | Validation failures, failed mutations, and abandonment on primary forms emit events or reach an error tracker (Sentry, Bugsnag, Honeybadger) with user-facing context | 🟡 silent — the team can't see where people give up |
| 10.5 Privacy and consent | No PII in event properties (use ids, not emails); consent respected where required (cookie/consent banner, `doNotTrack`, iOS ATT); retention documented | 🔴 PII or payment data in event payloads; 🟡 tracking fires before consent in a consent-required market |
| 10.6 Metrics are defined and owned | A doc or dashboard names the product metrics per flow (activation, step conversion, time-to-complete, error rate, retention) and who reads them | 🟡 events exist but no metric definitions; ⚪ if it lives in a dashboard the audit can't see |
| 10.7 Experimentation readiness | Feature flags or an A/B mechanism cover the primary flows, or the project states it doesn't need them yet | 🟡 none on a product that is actively iterating its flows; note (not a gap) for small/internal tools — say which |

```bash
# Evidence
jq '.dependencies + .devDependencies | keys' package.json 2>/dev/null | rg -i "posthog|mixpanel|amplitude|segment|analytics|gtag|plausible|rudder|heap|hotjar|clarity"
rg -n "ahoy|Ahoy" Gemfile app config 2>/dev/null | head
rg -n "\.track\(|\.capture\(|logEvent\(|gtag\(|plausible\(|ahoy\.track" src app | wc -l
rg -n "\.track\(|\.capture\(|logEvent\(" src app | rg -o "['\"][a-zA-Z_ .:-]+['\"]" | sort | uniq -c | sort -rn | head -30   # event names + convention
rg -n "Sentry|Bugsnag|Honeybadger|Rollbar" src app config Gemfile package.json | head
rg -ln "consent|cookie.*banner|doNotTrack|requestTrackingPermission" src app | head
rg -n "email:|user\.email|card|phone" $(rg -ln "\.track\(|\.capture\(" src app 2>/dev/null) 2>/dev/null | head   # PII in payloads
ls docs | rg -i "track|metric|analytics|kpi"
```

```ts
// ✅ correct — central definition, ids not PII, funnel-able
track("invoice_sent", { invoice_id, amount_cents, currency, step: "review" });

// ❌ wrong — literal at the call site, PII in payload, not funnel-able
posthog.capture("clicked send", { email: user.email, name: user.name });
```

When 10.1, 10.2, or 10.6 is 🟡, the recommendation is an **Instrumentation plan** (format: `references/output-format.md` § 4): for each primary flow, the events to emit (start / step / complete / fail, with properties), the metric each enables, the product decision that metric informs, and a candidate tool. **Tool rule:** name one or two candidates that fit the stack and scale *as of this skill's date*, then always add the line: *"Before choosing, survey the current market — options, pricing, hosting, and privacy terms change; these candidates are a starting point, not a decision."* If a web-search tool is available, one bounded search to confirm current options is allowed (`references/audit-workflow.md` → Phase 2b); installing, signing up, or adding keys is never allowed. Point at `assets/instrumentation-plan.example.md` as the tracking-plan starter.

## Grade ladder

- 🔴 **Blocker** — a primary flow can't be completed by some users, a WCAG level A criterion fails, or a primary viewport is broken. Any 🔴 ⇒ verdict **Needs work**.
- 🟡 **Gap** — usable, but costs users effort, trust, or excludes some of them (AA failures, inconsistency, missing states). Only 🟡 ⇒ **Acceptable with gaps**.
- 🟢 **Solid** — a concrete rule was checked and satisfied.
- ⚪ **Unverifiable from code** — always carries the exact URL, viewport, element, or question a human must check. ⚪ never changes the verdict but always appears in the report.

## Conservatism rule

A grade other than 🟢 must cite evidence: a path and line, a count from a command with its output, a screenshot from the browser pass, the flow walk (dimension 9), or a documented claim that turned out false. If you can't produce that, the check is either 🟢 (you checked and it's fine) or ⚪ (you couldn't check) — never 🟡 on taste. Never 🟢 something you didn't check — in particular contrast, focus visibility, and reflow without a runtime. A flow is "too complex" only against its own required data (9.x), and an analytics tool is never presented as the single answer (10.x) — always a dated candidate plus the market-survey line.

## Recommendation rule

Every check gets a `→ Recommendation:` line, regardless of grade:

- 🔴 / 🟡 — the concrete change: the file or component to edit, the pattern to apply (name the token, component, attribute, or i18n key), and, where useful, the one-line example. Imperative mood.
- 🟢 — one line starting with **Keep:** naming the practice that earned the grade, so a re-audit knows what not to regress.
- ⚪ — **Verify:** the exact action (URL + viewport + element, a DevTools panel, a question for the designer/PM) and what result would make it 🟢 vs. 🔴/🟡.
- Dimension 9 (🔴/🟡) — the recommendation is the **proposed step sequence**: which steps merge, move, or disappear, which fields get prefilled and from where, naming the files. "Simplify" alone is not a recommendation.
- Dimension 10 (🟡) — the recommendation is an **Instrumentation plan**: events + properties → metric → decision it informs → candidate tool → the market-survey line.

A check with a grade but no recommendation is incomplete; do not deliver the report until every check has one.

Config `.eagerworks/ui-ux-audit.json` may disable a dimension, set the WCAG level, name the primary flows and viewports, or forbid the browser pass — any disabled dimension is disclosed in the report, never silently omitted (`references/config.md`).
