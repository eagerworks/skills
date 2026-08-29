# ui-ux-audit

A portable agent skill that audits a project's **user interface and user experience** — navigation, design-system consistency, responsiveness, WCAG 2.2 AA accessibility, forms, feedback states, microcopy/i18n, perceived performance, **flow simplicity** (is a journey longer than its outcome requires, and the concrete shorter sequence), and **product instrumentation** (which flows and metrics to track to decide with data, and with which tool) — and returns a graded report in which **every item carries a concrete recommendation** of what to do. Graded from source, with an optional browser pass when a running app is available. Stack-agnostic, with Rails/Hotwire, React/Next, and React Native/Expo examples. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Discovery of the UI stack(s), design tokens, component library, i18n setup, and the product's **primary flows** — which are graded first and escalate defects
- Ten dimensions: information architecture & navigation, visual consistency & design system, layout & responsiveness, accessibility (WCAG 2.2 AA), forms & input, feedback & system states, content & microcopy, performance as experienced, flow simplicity, product instrumentation & metrics
- A **Flow map** per primary flow (steps · fields · decisions today → proposed) and recommendations that name the merged, deferred, or prefilled steps — "this flow is 4 screens for 1 required step; here is the 2-screen version"
- An **Instrumentation plan** per primary flow — events with properties → the metric they build → the product decision it informs — plus one or two candidate tools that fit the stack, an optional bounded web search to confirm they're current, and always the recommendation to survey the market before choosing; the audit never installs or configures a tool
- A four-level grade per check — 🔴 Blocker / 🟡 Gap / 🟢 Solid / ⚪ Unverifiable from code — rolled up to a mechanical verdict: Needs work / Acceptable with gaps / Solid
- A `→ Recommendation:` on **every** check: the file and pattern to change for 🔴/🟡, what to keep for 🟢, exactly how to verify for ⚪
- A **Work Plan**: every blocker and gap as an ordered task with effort (S/M/L), evidence, and the concrete change
- An optional, strictly bounded runtime pass: navigate, resize, tab through, screenshot, read computed styles and console — never start the app, install, submit real forms, or click destructive actions
- The report is printed in chat **and** saved to `docs/ui-ux-audit.md` in the audited project (the directory is created if needed) — the one file the skill writes; it never edits UI code, commits, or pushes
- A WCAG 2.2 quick checklist and a canonical UI-state matrix, plus a copyable state checklist to recommend

## Layout

```
SKILL.md                        # hub: discovery, the ten dimensions, grades, recommendation rule, gotchas (agent entrypoint)
references/
  rubric.md                     # full checklist per dimension, grade ladder, conservatism + recommendation rules
  audit-workflow.md             # phase-by-phase procedure, the bounded browser pass, the bounded market check, saving the report
  output-format.md              # the report markdown: scorecard, Work Plan, Flow map, Instrumentation plan, footer
  accessibility.md              # WCAG 2.2 AA checklist, contrast thresholds, keyboard test script
  ui-states.md                  # loading / empty / error / success / confirm / offline matrix per screen type
  flow-simplicity.md            # the flow walk, simplification patterns, when not to simplify, worked example
  instrumentation.md            # event sets per flow, metric → decision catalogue, privacy checklist, dated tool landscape
  config.md                     # .eagerworks/ui-ux-audit.json schema
assets/
  audit-report.md               # report template → docs/ui-ux-audit.md
  ui-states.example.md          # copyable state checklist to recommend when states are missing
  instrumentation-plan.example.md  # copyable tracking-plan starter to recommend when flows aren't instrumented
  ui-ux-audit.example.json      # starter config
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration required. To move the report, name the primary flows and viewports, point at a running app, change the WCAG level, forbid the browser pass or the web-search market check, hint at the analytics setup, or disable a dimension that doesn't apply, add `.eagerworks/ui-ux-audit.json` — see [`references/config.md`](references/config.md) and [`assets/ui-ux-audit.example.json`](assets/ui-ux-audit.example.json). Disabled dimensions are always disclosed in the report.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill ui-ux-audit
```
