---
name: ui-ux-audit
description: >-
  Audits a project's user interface and user experience — information architecture, visual consistency, responsiveness, WCAG 2.2 AA accessibility, forms, feedback states, microcopy/i18n, and perceived performance — from its source code (plus an optional runtime pass in a browser when a URL is available) and returns a graded report where every item carries a concrete recommendation of what to do. Use when asked "audit the UI/UX of this project", "review the usability / accessibility of this app", "is this frontend accessible / responsive / consistent", "UX review of the frontend / screens", "what should we fix in the UI", "how good is our design system usage", or to re-check after UI work was done. Works for Rails views/Hotwire, React/Next/Vue, and React Native/Expo. The report is printed in chat and saved to docs/ui-ux-audit.md in the audited project.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# UI/UX Audit Skill

A **UI/UX audit** grades how well a product's interface lets people do what they came to do: find the right screen, understand it, act on it, recover from mistakes, and get there on any device with any ability. This skill audits a project across eight dimensions, grades every check with evidence, and — the deliverable — attaches a **concrete recommendation to every single item**, then rolls the blockers and gaps into an ordered **Work Plan**.

The audit is **read-only with exactly one write**: the finished report is printed in chat **and** saved to `docs/ui-ux-audit.md` at the audited repo's root (`docs/` is created if missing; the file is overwritten on re-run so the project keeps one current audit). Nothing else is created, edited, committed, or pushed.

## Discovery — Do This First

Classify the UI before grading anything. Every check in `references/rubric.md` depends on knowing where screens, styles, and copy live.

**1. Detect the UI stack(s)** from manifests and directory layout:

| Signal | UI stack | Where the UI lives |
|---|---|---|
| `app/views/`, `app/components/`, `app/javascript/controllers/` | Rails — ERB/Slim, ViewComponent, Hotwire/Stimulus | `app/views/**`, `app/components/**`, `app/assets/stylesheets/`, `config/locales/` |
| `next.config.*`, `app/` or `pages/`, `*.tsx` | React / Next.js | `app/**`, `pages/**`, `components/**`, `styles/`, `tailwind.config.*` |
| `vite.config.*` + `src/App.*`, `*.vue`, `nuxt.config.*` | Vue / Nuxt / SPA | `src/views/`, `src/components/`, `src/router/` |
| `app.json` / `app.config.*`, `expo`, `react-native` in deps | React Native / Expo | `app/**` (Expo Router) or `src/screens/**`, `src/components/**` |
| `tailwind.config.*`, `*.module.css`, `styled-components`, `*.scss` | Styling approach | tokens, theme files, global styles |
| `@shadcn/ui`, `@mui/*`, `@radix-ui/*`, `@chakra-ui/*`, `react-native-paper`, `view_component` | Component library | how much of the UI is built on it vs. one-offs |

A monorepo may hold several UIs (web app + mobile app + marketing site) — audit each and grade the root on shared tokens/components.

**2. Inventory the UI surface** — read, don't skim: the route map (`config/routes.rb`, `app/**/page.tsx`, `src/router/*`, Expo Router tree), the layout/shell components, the design tokens (`tailwind.config.*`, `theme.ts`, `_variables.scss`, `app/assets/stylesheets/*`), the i18n setup (`config/locales/*`, `i18n.ts`, `locales/*.json`), shared form/button/modal/table components, empty/error/loading components, `README.md` and any design docs (`docs/design*`, `DESIGN.md`, Storybook `*.stories.*`), and existing a11y/UI tooling (`eslint-plugin-jsx-a11y`, `axe-core`, `erb_lint`, `pa11y`, Lighthouse CI, Playwright/Capybara system specs).

**3. Identify the primary flows** — the 3–6 journeys the product exists for (sign-up/login, the core create/edit object, checkout/submit, search/browse, settings). Source, in order: `.eagerworks/ui-ux-audit.json#primaryFlows` → README/product docs → infer from routes and nav. The audit grades those screens first and names them in the report; a blocker on a primary flow outranks anything on a secondary screen.

**4. Decide whether a runtime pass is possible.** If the user gave a URL, config sets `appUrl`, or a dev server is already listening, `references/audit-workflow.md` Phase 2 says what you may do in the browser. **Never start the server, install dependencies, or run setup** — without a running app, runtime-only checks (contrast, focus visibility, real viewport behaviour) are ⚪ with the exact URL/question needed.

## What This Skill Does NOT Do

It doesn't redesign anything, doesn't run user research, doesn't touch Figma, and can't judge brand fit, taste, or conversion impact — those need a designer, analytics, or users and are graded ⚪ **Unverifiable from code** with the exact question to ask. It never edits UI files: every recommendation *describes* the change.

## The Eight Dimensions

| # | Dimension | A good UI needs… |
|---|---|---|
| 1 | Information architecture & navigation | A clear route map, persistent primary nav, working back/deep links, no dead ends, proper 404/403 screens |
| 2 | Visual consistency & design system | Tokens for color/spacing/type used everywhere; reusable components instead of one-offs; one icon set; theme parity |
| 3 | Layout & responsiveness | Primary viewports handled, no overflow, touch targets ≥ 44 px, safe areas, sensible breakpoints |
| 4 | Accessibility (WCAG 2.2 AA) | Semantic markup, labels and alt text, keyboard reachability, visible focus, contrast, reduced motion, `lang` |
| 5 | Forms & input | Labels, inline validation with clear placement, correct input types, submit states, data preserved on error |
| 6 | Feedback & system states | Loading, empty, error, success, offline, permission-denied states; confirmations for destructive actions; undo |
| 7 | Content & microcopy | Consistent tone, action-verb buttons, no jargon, i18n-ready strings, pluralization, localized dates/numbers |
| 8 | Performance as experienced | No layout shift sources, sized images, font loading strategy, long lists paginated/virtualized, no jank sources |

Full checks, decision rules, and Rails / React / React Native examples: `references/rubric.md` — read it before grading, it is the authoritative checklist.

## Grades, Verdict, and the Recommendation Rule

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | Users can't complete a primary flow, a WCAG level A criterion fails, or a primary viewport is broken |
| 🟡 **Gap** | It works, but hurts usability, consistency, or WCAG AA — costs users effort or trust |
| 🟢 **Solid** | Checked against a concrete rule and clean |
| ⚪ **Unverifiable from code** | Needs a running app, real users, analytics, or a designer's judgement |

Verdict: **Needs work** (any 🔴) → **Acceptable with gaps** (🟡 only) → **Solid**. ⚪ never moves it.

**Every item gets a recommendation.** In the findings, each check — 🔴, 🟡, 🟢 *and* ⚪ — ends with a `→ Recommendation:` line. 🔴/🟡 name the file or component to change and the pattern to apply; 🟢 say what to keep doing (one line); ⚪ say exactly how to verify. A grade without a recommendation is an incomplete finding. The **Work Plan** then orders every 🔴 and 🟡 with effort (S/M/L) and evidence. Format: `references/output-format.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The eight dimensions in full, grade ladder, conservatism and recommendation rules | `references/rubric.md` |
| Running the audit end-to-end; the optional browser pass and what's forbidden; saving the report | `references/audit-workflow.md` |
| The exact report markdown, Work Plan table, per-item recommendation format | `references/output-format.md` |
| WCAG 2.2 AA quick checklist, contrast thresholds, keyboard test script — what dimension 4 grades against | `references/accessibility.md` |
| The canonical state matrix (loading / empty / error / success / offline / denied) — what dimensions 5–6 grade against | `references/ui-states.md` |
| The optional `.eagerworks/ui-ux-audit.json` config | `references/config.md` |

Copyable assets live in `assets/`:
- `assets/audit-report.md` — the report template; fill it in and save it as `docs/ui-ux-audit.md`
- `assets/ui-states.example.md` — a state-matrix checklist to recommend when dimension 6 is 🔴
- `assets/ui-ux-audit.example.json` — starter config

## Critical Gotchas

1. **One write, nothing else.** The only file you create is the report at `docs/ui-ux-audit.md` (or `reportPath` from config). Never edit a view, component, stylesheet, or locale; never `git commit`, `git push`, or open a PR. Recommendations *describe* fixes — they don't apply them.

2. **Never start, install, or submit.** No `rails s`, `npm run dev`, `expo start`, `npm install`, `bin/setup`. In a browser pass you navigate, resize, tab, screenshot, and read the console — you never submit a form that creates real data, never log in with credentials you weren't given, never click destructive actions.

3. **A linter in `package.json` is not a passing linter.** `eslint-plugin-jsx-a11y` configured ≠ a11y clean. Run it only if it's already installed and non-interactive (`references/audit-workflow.md`); otherwise grade from source and say so.

4. **Don't grade what you can't see 🟢.** Contrast ratios, visible focus rings, actual reflow at 375 px, and animation jank need a running app or computed values. Without them: ⚪ with the URL/viewport/element to check — never a guessed 🟢.

5. **Evidence is a path, a count, or a screenshot — not a taste.** "Feels inconsistent" is not a finding; `grep -rn "#[0-9a-fA-F]\{6\}" app/views | wc -l` → 47 hardcoded colors across 12 files is. Every non-🟢 grade cites `file:line`, a command with output, or a screenshot.

6. **Recommendations name the file.** "Improve form errors" is not a recommendation; "In `app/views/users/_form.html.erb`, render `f.object.errors[:email]` under the field with `aria-describedby`, not only in the flash" is.

7. **Zero blockers is a valid result.** Don't pad the Work Plan with preferences; a row must trace to a 🔴 or 🟡. 🟢 items still get their one-line "Keep:" recommendation in the findings.

8. **Print the whole report in chat, then save the identical text.** The user asked for both; "see the file" is not the deliverable.
