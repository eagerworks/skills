# Accessibility Checklist — What Dimension 4 Grades Against

WCAG 2.2 level AA is the default bar (`wcagLevel` in config can lower it to A or raise it to AAA). Level A failures are 🔴 on any screen; AA failures are 🟡, escalated to 🔴 on a primary flow when they block completion.

## Quick checklist by success criterion

| Area | Criterion | Level | Pass looks like | Fails as |
|---|---|---|---|---|
| Images | 1.1.1 Non-text content | A | Meaningful `alt` / `accessibilityLabel`; decorative `alt=""` or `aria-hidden="true"` | 4.2 |
| Structure | 1.3.1 Info & relationships | A | Headings in order, lists as `<ul>/<ol>`, tables with `<th scope>`, form groups in `<fieldset>` | 4.1 |
| Color | 1.4.1 Use of color | A | Errors/status carry an icon or text, not color alone | 4.7 / 5.2 |
| Contrast | 1.4.3 Contrast (minimum) | AA | Body text ≥ 4.5:1; large text (≥ 24 px, or ≥ 18.66 px bold) ≥ 3:1 | 4.6 |
| Contrast | 1.4.11 Non-text contrast | AA | Input borders, focus rings, icon buttons ≥ 3:1 against adjacent color | 4.6 |
| Reflow | 1.4.10 Reflow | AA | No 2-D scroll at 320 px CSS width (≈ 400 % zoom on 1280) | 3.5 |
| Zoom | 1.4.4 Resize text | AA | Usable at 200 %; no `maximum-scale=1` / `user-scalable=no` | 3.5 |
| Spacing | 1.4.12 Text spacing | AA | Nothing clipped when line-height 1.5×, paragraph 2×, letter 0.12em | 3.5 |
| Keyboard | 2.1.1 Keyboard | A | Everything operable by keyboard; custom widgets handle Enter/Space/arrows | 4.4 |
| Keyboard | 2.1.2 No keyboard trap | A | Focus can leave every component; modals release on Escape/close | 4.4 |
| Focus | 2.4.3 Focus order | A | Tab order matches visual order; no positive `tabindex` | 4.4 |
| Focus | 2.4.7 Focus visible | AA | Visible indicator on every focused element | 4.5 |
| Focus | 2.4.11 Focus not obscured (minimum) | AA | Focused element not fully hidden behind sticky bars | 3.6 / 4.5 |
| Targets | 2.5.8 Target size (minimum) | AA | ≥ 24×24 CSS px, or 24 px spacing between targets (44 recommended) | 3.3 |
| Motion | 2.3.3 Animation from interactions | AAA (graded as 🟡) | `prefers-reduced-motion` respected | 4.7 |
| Titles | 2.4.2 Page titled | A | Unique `<title>` / screen title per page | 1.6 |
| Skip | 2.4.1 Bypass blocks | A | Skip link or landmarks | 4.1 |
| Language | 3.1.1 Language of page | A | `<html lang="…">` | 4.7 |
| Labels | 3.3.2 Labels or instructions | A | Visible label for every input | 4.3 / 5.1 |
| Errors | 3.3.1 Error identification | A | Error text names the field and is associated (`aria-describedby`) | 5.2 |
| Errors | 3.3.3 Error suggestion | AA | Error says how to fix it | 5.3 |
| Redundant entry | 3.3.7 Redundant entry | A | Multi-step forms don't re-ask what the user already entered | 5.6 |
| Auth | 3.3.8 Accessible authentication | AA | No cognitive test; password managers and paste allowed | 5.4 |
| Name/role | 4.1.2 Name, role, value | A | Custom widgets expose role, name, state (`aria-expanded`, `aria-selected`) | 4.1 / 4.2 |
| Status | 4.1.3 Status messages | AA | Toasts/counters use `role="status"` / `aria-live="polite"` | 4.7 / 6.4 |

## Contrast thresholds

| Text | Minimum (AA) | Enhanced (AAA) |
|---|---|---|
| Normal text (< 24 px, or < 18.66 px bold) | 4.5:1 | 7:1 |
| Large text | 3:1 | 4.5:1 |
| UI component boundary, focus ring, icon | 3:1 | — |
| Disabled controls, logos, decorative | exempt | — |

Compute from tokens when possible: hex pairs can be checked with any contrast formula (relative luminance per WCAG); `oklch()`/`hsl()` with alpha, gradients, and images-behind-text need the browser — grade ⚪ with the exact pair to check.

## Keyboard test script (runtime pass)

Run on each primary-flow screen:

1. Load the page. Press Tab once — a skip link or the first nav item should receive visible focus.
2. Tab through the whole page. Note any interactive element skipped (custom `div` buttons), any element focused but invisible, any order jump.
3. Open every overlay (menu, modal, drawer, date picker). Tab: focus stays inside. Escape: closes and returns focus to the trigger.
4. Activate a button with Space and a link with Enter. Operate any custom select/tabs/slider with arrow keys.
5. Submit a form empty (only on a disposable environment, never with real side effects). Focus should move to the first error or an error summary; the error is announced.
6. Zoom to 200 % and set the window to 320 px wide. No horizontal scroll; nothing clipped.

Each step maps to a check: 1 → 4.1/4.4, 2 → 4.4/4.5, 3 → 4.4, 4 → 4.1, 5 → 5.2, 6 → 3.5.

## Stack notes

**Rails/ERB**: `f.label` before every `f.*_field`; `image_tag` needs `alt:`; use `button_to` for actions (not `link_to` with `method:`); `erb_lint` with `Rails::Accessibility`-style custom linters if present. Hotwire: Turbo Frame swaps should move focus (`autofocus` on the new heading or `data-turbo-action`).

**React/Next**: `eslint-plugin-jsx-a11y` in `recommended` mode catches 4.2/4.3/4.4 statically; Radix/shadcn primitives ship correct roles and focus management — a 🟢 on 4.4 is justified when overlays come from them. Custom `onClick` on `div` is a 4.1 fail unless it also has `role="button"`, `tabIndex={0}`, and key handlers.

**React Native/Expo**: `accessibilityRole` + `accessibilityLabel` on every `Pressable`/`TouchableOpacity`; `accessibilityState` for toggles; `AccessibilityInfo.announceForAccessibility` for status; `hitSlop` to reach 44 pt targets; `allowFontScaling` must not be globally `false`; `accessibilityViewIsModal` on custom modals. Check images via `accessible`/`accessibilityLabel` or `accessibilityElementsHidden`.

## Grading shortcuts

| Situation | Grade |
|---|---|
| Global `outline: none` / `focus:outline-none` with no `focus-visible` replacement | 🔴 4.5 |
| Clickable `div`/`span` is the only trigger for a primary-flow action | 🔴 4.1 |
| `<img>` with no `alt` attribute at all in content views | 🟡 4.2 (🔴 if the image *is* the content, e.g. a chart with no text alternative on a primary screen) |
| Inputs with placeholder only, primary form | 🔴 4.3 |
| `user-scalable=no` in the viewport meta | 🔴 3.5 |
| Modals from Radix/shadcn/`<dialog>`/Headless UI with no custom overrides | 🟢 4.4 for focus trapping |
| Tokens defined but contrast not computable from source, no runtime | ⚪ 4.6 with the pairs to test |
| `jsx-a11y` installed, `recommended`, and the lint passes when run | 🟢 4.2, 4.3 (still check 4.5/4.6 separately — the linter can't) |
