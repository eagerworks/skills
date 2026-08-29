# ui-ux-audit

A portable agent skill that audits a project's **user interface and user experience** — navigation, design-system consistency, responsiveness, WCAG 2.2 AA accessibility, forms, feedback states, microcopy/i18n, and perceived performance — and returns a graded report in which **every item carries a concrete recommendation** of what to do. Graded from source, with an optional browser pass when a running app is available. Stack-agnostic, with Rails/Hotwire, React/Next, and React Native/Expo examples. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Discovery of the UI stack(s), design tokens, component library, i18n setup, and the product's **primary flows** — which are graded first and escalate defects
- Eight dimensions: information architecture & navigation, visual consistency & design system, layout & responsiveness, accessibility (WCAG 2.2 AA), forms & input, feedback & system states, content & microcopy, performance as experienced
- A four-level grade per check — 🔴 Blocker / 🟡 Gap / 🟢 Solid / ⚪ Unverifiable from code — rolled up to a mechanical verdict: Needs work / Acceptable with gaps / Solid
- A `→ Recommendation:` on **every** check: the file and pattern to change for 🔴/🟡, what to keep for 🟢, exactly how to verify for ⚪
- A **Work Plan**: every blocker and gap as an ordered task with effort (S/M/L), evidence, and the concrete change
- An optional, strictly bounded runtime pass: navigate, resize, tab through, screenshot, read computed styles and console — never start the app, install, submit real forms, or click destructive actions
- The report is printed in chat **and** saved to `docs/ui-ux-audit.md` in the audited project (the directory is created if needed) — the one file the skill writes; it never edits UI code, commits, or pushes
- A WCAG 2.2 quick checklist and a canonical UI-state matrix, plus a copyable state checklist to recommend

## Layout

```
SKILL.md                        # hub: discovery, the eight dimensions, grades, recommendation rule, gotchas (agent entrypoint)
references/
  rubric.md                     # full checklist per dimension, grade ladder, conservatism + recommendation rules
  audit-workflow.md             # phase-by-phase procedure, the bounded browser pass, saving the report
  output-format.md              # the report markdown: scorecard, Work Plan, per-item recommendations, footer
  accessibility.md              # WCAG 2.2 AA checklist, contrast thresholds, keyboard test script
  ui-states.md                  # loading / empty / error / success / confirm / offline matrix per screen type
  config.md                     # .eagerworks/ui-ux-audit.json schema
assets/
  audit-report.md               # report template → docs/ui-ux-audit.md
  ui-states.example.md          # copyable state checklist to recommend when states are missing
  ui-ux-audit.example.json      # starter config
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

Zero configuration required. To move the report, name the primary flows and viewports, point at a running app, change the WCAG level, forbid the browser pass, or disable a dimension that doesn't apply, add `.eagerworks/ui-ux-audit.json` — see [`references/config.md`](references/config.md) and [`assets/ui-ux-audit.example.json`](assets/ui-ux-audit.example.json). Disabled dimensions are always disclosed in the report.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill ui-ux-audit
```
