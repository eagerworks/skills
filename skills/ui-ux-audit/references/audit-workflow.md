# UI/UX Audit — Workflow

End-to-end procedure. Read `references/rubric.md` for what each check means; this file is about *how* to run the audit safely and what to do with the result.

## Phase 0 — Config

Look for `.eagerworks/ui-ux-audit.json` at the repo root (`references/config.md`). It can move the report, name the primary flows and viewports, set the WCAG level, disable dimensions, point at a running app (`appUrl`), or forbid the browser pass. Note anything it changes — it goes in the report's disclosure line.

## Phase 1 — Discovery

1. Detect the UI stack(s) and any monorepo split (`SKILL.md` → Discovery).
2. Read the UI surface in full: route map, layout/shell components, design tokens, i18n setup, shared primitives (`components/ui/*`, `app/components/*`, `app/views/shared/*`), state components (loading/empty/error), `README.md`, design docs, Storybook stories, and existing UI tooling configs (`.eslintrc*` for `jsx-a11y`, `.erb-lint.yml`, `lighthouserc.*`, `playwright.config.*`, `spec/system/*`).
3. Identify the **primary flows** (config → docs → inferred from routes/nav) and list the screens each one touches. Write this list down — it goes in the report header and decides 🔴 vs 🟡 for several checks.
4. Build the **screen inventory**: for each primary-flow screen, its file(s), the form/list/detail pattern it follows, and which state components it uses. This is what you grade first.
5. Do the **flow walk** for each primary flow (`references/flow-simplicity.md`): steps in order, data collected per step, whether the outcome requires it (model validations / API schema), whether the system already knows it, decisions per screen. It is the only evidence dimension 9 accepts.
6. Take the **analytics inventory** (`references/instrumentation.md`): tracker dependencies and whether one is initialized in the shell, the event call sites and their names, an error tracker, consent handling, feature flags, and any tracking plan / metrics doc under `docs/`. Note what `.eagerworks/ui-ux-audit.json#analytics` claims and verify it.

```bash
git rev-parse --show-toplevel
git rev-parse --short HEAD; git branch --show-current
bin/rails routes 2>/dev/null | head -80                     # Rails (read-only)
find app pages src -name "page.tsx" -o -name "*.vue" -o -name "_layout.tsx" 2>/dev/null | head -80
ls config/locales locales src/locales 2>/dev/null
cat tailwind.config.* theme.ts src/theme/* 2>/dev/null | head -120
jq '.dependencies + .devDependencies | keys' package.json 2>/dev/null | rg -i "a11y|axe|lighthouse|storybook|shadcn|radix|mui|chakra|paper|reanimated|flash-list"
jq '.dependencies | keys' package.json 2>/dev/null | rg -i "posthog|mixpanel|amplitude|segment|gtag|plausible|sentry|launchdarkly|flags"
rg -n "ahoy|flipper|sentry" Gemfile 2>/dev/null; ls docs 2>/dev/null | rg -i "track|metric|analytics"
```

## Phase 2 — The optional runtime pass (the only interactive step)

The audit is graded from source. A runtime pass **adds** evidence for checks that need a rendered UI (3.1, 3.2, 3.6, 4.5, 4.6, 6.1, 7.6, 8.4). It happens only if **one** of these is true:

- the user gave a URL in the request, or
- config sets `appUrl`, or
- a dev server is **already listening** (`lsof -iTCP -sTCP:LISTEN -P | rg "3000|5173|8081"`) and the user hasn't said not to use it.

You may, in the browser (Claude in Chrome, Playwright MCP, or an equivalent):

- navigate to the primary-flow screens you can reach without credentials, or with credentials the user explicitly provided for this audit;
- resize the viewport to the configured widths (defaults: 375, 768, 1280) and screenshot each primary screen;
- press Tab through a screen and note focus order and visibility; press Escape on overlays;
- read computed styles (contrast, target size) via the page's DevTools/JS evaluation;
- read console errors and network failures on load;
- toggle `prefers-reduced-motion` / `prefers-color-scheme` emulation.

You must **never**:

- start the app (`rails s`, `npm run dev`, `expo start`, `docker compose up`), install dependencies, or run setup/migrations — no runtime ⇒ those checks are ⚪;
- submit a form that creates, changes, or deletes real data; the only submits allowed are ones the user explicitly sanctioned on a disposable environment;
- click destructive actions (delete, archive, cancel subscription), even to test the confirmation — grade the confirmation from source;
- enter credentials you weren't given, bypass auth, or visit production admin surfaces;
- run a Lighthouse/axe CLI that isn't already installed in the project. If `axe-core`/`@axe-core/cli`, `pa11y`, or `lighthouse` **is** installed, you may run it once against the URL, non-interactively, with a timeout.

```bash
# ✅ correct — already installed, read-only, bounded
timeout 120 npx --no-install @axe-core/cli http://localhost:3000/ 2>&1 | tail -40
timeout 120 npx --no-install eslint --ext .tsx src --rule '{"jsx-a11y/alt-text":"error"}' 2>&1 | tail -20

# ❌ wrong
npm run dev &                     # starting the app
npx axe http://localhost:3000     # installs on the fly
curl -X POST http://localhost:3000/projects   # mutates data
```

Record for every screenshot: URL, viewport, what it shows. Screenshots are evidence — reference them by filename in the report (save them in the scratchpad, not in the repo). Every check you *would* have verified at runtime but couldn't becomes ⚪ with the exact URL + viewport + element to check.

## Phase 2b — The optional market check (web search only)

Dimension 10 recommends analytics tooling. The candidates in `references/instrumentation.md` are a **dated snapshot**; the market changes. If the agent has a web-search tool and config doesn't set `marketCheck: false`:

- run **at most two** searches, scoped to the stack (e.g. `product analytics tools 2026 React Native`, `self-hosted product analytics Rails 2026`);
- use the results only to confirm, add, or drop candidates in the Instrumentation plan, and cite them there;
- list the exact queries in the report footer (`web searches: …`).

If no search tool is available (or `marketCheck: false`), use the reference table and say in the plan that it is a snapshot as of the skill's date. In every case the plan ends with the market-survey line — the audit **never** signs up for, installs, configures, or adds keys for any tool.

```bash
# ❌ wrong — the audit never does these
npm install posthog-js
bundle add ahoy_matey
curl https://app.posthog.com/signup
```

## Phase 3 — Grade

Walk `references/rubric.md` dimension by dimension, primary-flow screens first. For each check write the grade, its evidence, **and its recommendation** as you go — the recommendation is not a separate pass, it's part of the finding. Roll each dimension up to its worst check. Disabled dimensions get one line: `_Dimension N disabled by config_`.

Apply the primary-flow escalation: the same defect is one level harsher on a primary-flow screen when it stops or misleads the user (rubric tables say where).

## Phase 4 — Write the Work Plan

Turn every 🔴 and 🟡 into a task, ordered:

1. All 🔴, primary-flow items first, then in the order a designer/developer would naturally fix them (structure before styling: navigation → semantics → forms → states).
2. All 🟡, ordered by leverage — dimension 2 (tokens/components: fixing the source fixes every screen) and 4 (accessibility) first, then 6, 9 (a shorter flow removes whole screens of defects), 5, 3, 1, 7, 8, and 10 last (instrumentation is how the team will *see* the effect of the rest — it goes in the plan, but never ahead of a defect users hit today).

Each task: `#`, dimension, grade, the concrete change (one sentence, imperative, naming the file/component to create or edit and the pattern to apply), effort **S** (< 1 h) / **M** (half a day) / **L** (more), evidence. Point at `assets/ui-states.example.md` for dimension 6 blockers, at `references/accessibility.md` for dimension 4, and at `assets/instrumentation-plan.example.md` for dimension 10. A dimension 9 task states the proposed step sequence; a dimension 10 task says "instrument <flow> per the Instrumentation plan (§ 10)" rather than repeating the table. Don't invent tasks that don't trace to a graded check. Merge tasks that share a fix (e.g. "adopt `<Button>` everywhere" covers several 2.3 findings) and list every evidence location in the row.

## Phase 5 — Deliver: chat first, then the file

1. Fill `assets/audit-report.md` (format: `references/output-format.md`). Check that **every** check in the findings has a `→ Recommendation:` line — 🟢 and ⚪ included — that § 9 has a Flow map row per primary flow, and that § 10 has an Instrumentation plan (or a **Keep:** line when the flows are already instrumented) ending with the market-survey line.
2. **Print the complete report in chat** — the whole thing, not a summary.
3. Save the identical markdown:

```bash
mkdir -p docs
# write the report to docs/ui-ux-audit.md (or reportPath from config)
```

Overwrite an existing report; the project keeps one current audit. Do **not** `git add`, commit, or push it — tell the user it's there and unstaged. If the project's `.gitignore` ignores `docs/`, say so instead of working around it.

## Re-auditing

When asked to re-check after UI work was done, run the full audit again (don't diff the old report — a fix on one screen can regress a token or a shared component) and, if a previous report exists, add a short **Since last audit** section listing checks that changed grade.
