# UI/UX Audit — Output Format

The same markdown is printed in chat and saved to `docs/ui-ux-audit.md`. Fill `assets/audit-report.md`; every section below is required, in this order.

## 1. Header

```markdown
# UI/UX Audit — <repo name>

- **Date:** YYYY-MM-DD
- **Commit:** `<short sha>` on `<branch>`
- **UI stack:** Rails 7.1 ERB + Hotwire, Tailwind 3 / Next.js 14 + shadcn/ui / Expo SDK 51
- **Primary flows:** Sign-up → onboarding · Create project · Invite member · Checkout
- **Runtime pass:** yes — `http://localhost:3000` at 375 / 768 / 1280 px | no — graded from source only
- **Verdict:** 🔴 Needs work | 🟡 Acceptable with gaps | 🟢 Solid
- **Blockers:** N · **Gaps:** N · **Unverifiable:** N
```

Then one paragraph — three sentences at most — saying what a user would trip over first and what the single highest-leverage fix is.

## 2. Scorecard

One row per dimension, worst check wins:

```markdown
| # | Dimension | Grade | Worst check | Evidence |
|---|---|---|---|---|
| 1 | Information architecture & navigation | 🟢 | — | `config/routes.rb`, `app/views/layouts/application.html.erb:14-38` |
| 2 | Visual consistency & design system | 🟡 | 2.2 47 hardcoded hex colors in 12 views | `grep -rnoE "#[0-9a-fA-F]{6}" app/views` |
| 4 | Accessibility | 🔴 | 4.3 checkout card fields unlabeled | `app/views/checkout/_card.html.erb:8-21` |
...
```

## 3. Work Plan

**This is the deliverable** — the ordered list of UI/UX work. Blockers first (primary flows first), then gaps by leverage (`references/audit-workflow.md` → Phase 4).

```markdown
| # | Dim | Grade | Task | Effort | Evidence |
|---|---|---|---|---|---|
| 1 | 4 | 🔴 | In `app/views/checkout/_card.html.erb`, wrap each card field in `f.label` + `f.text_field` with `autocomplete="cc-number"` / `cc-exp` / `cc-csc` and add `aria-describedby` pointing at the inline error `<p id="card_number_error">` | S | `_card.html.erb:8-21`, no `<label>` |
| 2 | 6 | 🔴 | Add an `ErrorBoundary` at `src/app/(app)/layout.tsx` and an `error.tsx` per route segment with a "Try again" button; today a failed `/api/projects` fetch renders a blank screen | M | `src/app/(app)/projects/page.tsx:22` unguarded `await` |
| 3 | 2 | 🟡 | Replace the 47 literal colors in `app/views/**` with the Tailwind theme tokens already defined in `tailwind.config.js` (`brand-*`, `danger`, `muted`); start with the 5 distinct reds used for errors | M | grep count 47 across 12 files |
```

Effort: **S** < 1 h · **M** ~ half a day · **L** more than a day.

## 4. Findings by dimension

For each dimension, **every check** with its grade, evidence (non-🟢), and a `→ Recommendation:` line. Recommendations are mandatory on every grade, including 🟢 and ⚪ — see `references/rubric.md` → Recommendation rule. Keep each finding to: grade, check id, one-sentence problem, evidence, recommendation.

```markdown
### 4. Accessibility (WCAG 2.2 AA) — 🔴

- 🔴 **4.3 Unlabeled inputs on checkout** — card number, expiry, and CVC are `<input placeholder="…">` with no label. Evidence: `app/views/checkout/_card.html.erb:8-21`.
  → Recommendation: use `f.label :card_number, "Card number"` + `f.text_field :card_number, autocomplete: "cc-number", inputmode: "numeric", "aria-describedby": "card_number_error"`; repeat for expiry (`cc-exp`) and CVC (`cc-csc`). Keep the placeholder as a format hint only.
- 🟡 **4.7 No reduced-motion handling** — `app/assets/stylesheets/animations.css` defines 6 keyframe animations with no `prefers-reduced-motion` guard. Evidence: `grep -rn "prefers-reduced-motion" app/assets` → none.
  → Recommendation: wrap non-essential animations in `@media (prefers-reduced-motion: no-preference)` in `animations.css`, or add Tailwind's `motion-safe:` prefix to the `animate-*` utilities in `app/views/shared/_toast.html.erb`.
- 🟢 **4.1 Semantic structure** — layout uses `header/nav/main/footer`, one `h1` per page.
  → Recommendation: Keep: the `<main>` landmark in `application.html.erb` and the `PageHeader` component that owns the `h1`.
- 🟢 **4.2, 4.4**
  → Recommendation: Keep: icon buttons go through `IconButton` (which requires `label`); modals use `<dialog>` with native focus trapping.
- ⚪ **4.6 Contrast** — tokens are defined as `oklch()` in `tailwind.config.js`; not computed without a browser.
  → Recommendation: Verify: open `/projects` at 1280 px, run the axe DevTools panel (or `npx --no-install @axe-core/cli http://localhost:3000/projects`) and check `text-muted` on `bg-card` ≥ 4.5:1 and the primary button text ≥ 4.5:1. Below 4.5:1 → 🟡; below 3:1 → 🔴.
```

Consecutive 🟢 checks may share one line and one **Keep:** recommendation when the same practice earned them; every 🔴, 🟡, and ⚪ gets its own entry.

## 5. Unverifiable from code

Every ⚪ collected in one place, each with the exact check for a human (it may repeat the findings entry — this section is the checklist someone runs with the app open):

```markdown
- ⚪ **3.1 Reflow at 375 px** — no runtime; `app/views/projects/index.html.erb` uses a 5-column table. Verify: open `/projects` at 375 px; if columns overflow the viewport → 🟡 (🔴 if the row actions are unreachable).
- ⚪ **8.7 Measurement** — no Lighthouse/Web Vitals config in the repo. Ask: is performance monitored in a dashboard (Sentry, Vercel Analytics, Datadog RUM)? If yes, link it in `README.md` → 🟢.
```

## 6. Footer

```markdown
---
_Generated by the `ui-ux-audit` skill · dimensions disabled by config: none · runtime pass: `http://localhost:3000` at 375/768/1280 px, 6 screenshots · commands executed: `npx --no-install @axe-core/cli http://localhost:3000/`_
```

The footer must list every command the audit actually **ran** and every URL it visited (or "none"), plus any dimension disabled via config, so the reader knows the audit's blast radius and blind spots.

## Rules

- Verdict is mechanical: any 🔴 → Needs work; else any 🟡 → Acceptable with gaps; else Solid. ⚪ never moves it.
- Work Plan rows map 1:1 to 🔴/🟡 checks (rows may merge checks that share one fix — list all their evidence) — no extra rows.
- Every check in the findings has a `→ Recommendation:` line; no finding without evidence (`references/rubric.md` → Conservatism rule).
- Chat output is the full report, then a one-line note: `Saved to docs/ui-ux-audit.md (unstaged).`
