# UI/UX Audit — Configuration

`.eagerworks/ui-ux-audit.json`, at the audited repo's root, is **entirely optional**. The audit works with no config — it discovers the UI from the repo, infers the primary flows, and writes the report to `docs/ui-ux-audit.md`. Add the file only to move the report, name the primary flows and viewports explicitly, point at a running app, change the WCAG level, forbid the browser pass, or disable a dimension that genuinely doesn't apply.

## Resolution order

`.eagerworks/ui-ux-audit.json` → statements in `README.md` / design docs / `AGENTS.md` (e.g. "the app is desktop-only, internal") → the skill's built-in defaults. A later source only fills in what an earlier one didn't set.

## Schema

All fields optional.

```jsonc
{
  // Where the report is written, relative to the repo root. Overwritten on every run.
  "reportPath": "docs/ui-ux-audit.md",

  // The journeys the product exists for. Screens on these flows are graded first
  // and defects on them are escalated (🟡 → 🔴 when they stop the user).
  // Each entry: a name and the routes/screens it touches (paths or component names).
  "primaryFlows": [
    { "name": "Sign up and onboard", "screens": ["/signup", "/onboarding/*"] },
    { "name": "Create project", "screens": ["/projects/new", "ProjectForm"] }
  ],

  // A running instance the audit may open in a browser for the runtime pass.
  // The audit never starts a server; if this is unreachable, runtime checks are ⚪.
  "appUrl": "http://localhost:3000",

  // Viewport widths (CSS px) for the runtime pass and for 3.x grading. RN apps:
  // logical widths of the target devices.
  "viewports": [375, 768, 1280],

  // WCAG 2.2 conformance level to grade dimension 4 against: "A" | "AA" | "AAA".
  "wcagLevel": "AA",

  // Whether the audit may open the app in a browser at all (navigate, resize,
  // screenshot, tab, read computed styles). false ⇒ graded from source only,
  // runtime-dependent checks become ⚪.
  "runtimePass": true,

  // Locales the product ships. One entry ⇒ 7.4 hardcoded strings are 🟢 by policy
  // (still graded on centralization). Omit to infer from the i18n setup.
  "locales": ["en", "es"],

  // Hints for dimension 10. Input, not evidence — the audit verifies each claim.
  "analytics": {
    "tool": "posthog",                       // what the team believes is wired
    "trackingPlan": "docs/tracking-plan.md", // where event names/metrics are defined
    "consentRequired": true                  // the product serves consent-required markets
  },

  // Whether the audit may run 1–2 web searches to confirm current analytics tool
  // options (only if the agent has a search tool). false ⇒ dated reference table only.
  // The audit never installs, signs up for, or configures a tool either way.
  "marketCheck": true,

  // Disable a dimension that doesn't apply (e.g. performance for an internal
  // tool by policy). Always disclosed in the report footer — never a silent omission.
  "dimensions": {
    "navigation":     { "enabled": true },
    "designSystem":   { "enabled": true },
    "responsiveness": { "enabled": true },
    "accessibility":  { "enabled": true },
    "forms":          { "enabled": true },
    "feedback":       { "enabled": true },
    "content":        { "enabled": true },
    "performance":    { "enabled": true },
    "flows":          { "enabled": true },
    "instrumentation": { "enabled": true }
  }
}
```

## Rules

- A disabled dimension appears in the footer as `dimensions disabled by config: performance` and is excluded from the scorecard and verdict.
- `primaryFlows` is input, not evidence — the audit still verifies those routes exist; a configured flow whose screens can't be found is reported as such.
- `runtimePass: false` never downgrades a check to 🟡; it turns runtime checks ⚪ with "set `runtimePass: true` and `appUrl`, or check `<screen>` at `<viewport>`" as the action.
- `analytics.*` is input, not evidence — a claimed `tool` that isn't a dependency initialized in the shell is reported as 🟡 10.1 with the discrepancy named; a `trackingPlan` path that doesn't exist is reported as such.
- `marketCheck: false` never changes a grade; it only removes the web-search step and makes the Instrumentation plan say its tool list is a dated snapshot. The market-survey line is always present.
- `wcagLevel: "A"` lowers the bar for 🟡 (AA-only criteria become notes in the recommendation, not gaps); `"AAA"` raises it. The default AA is what most legal and procurement requirements expect.

Starter: `assets/ui-ux-audit.example.json`.
