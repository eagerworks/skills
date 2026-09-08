# Design QA Test Cases — Output Format

## Report Language

Every template on this page is shown in English — the built-in default. When
`testCases.language` resolves to something else (`references/config.md`), translate every
human-facing string — headings, labels, case bodies, disclosure lines — while keeping
markdown structure, section order, and `file:line` citations byte-identical to what the
English template would produce.

**Never translated:** the `### TEST_CASES` sentinel and its JSON keys/enums (`title`,
`verifies`, `lens`, `priority`, `automate`, `level`, `framework`, `status`, `steps`,
`expected`), and the literal status values `New` / `covered`. Only the prose *values*
inside those fields follow `testCases.language`.

**The language of the conversation is never an input** — same invariant as `pr-review`'s
`review.language`. A QA chatting in Spanish still gets an English plan unless config or the
repo's `AGENTS.md`/`CLAUDE.md` says otherwise.

## Markdown Report (Default)

```markdown
## QA Test Cases — Checkout coupons

**Source:** LIN-482 (Linear) · **Design:** Figma MCP, 3 frames · **Code scanned:** 14 files
**Coverage:** 6 P0, 9 P1, 4 P2 · 5 already covered by automated tests

### P0 — Must pass before release

| Case | Verifies | Type | Automate | Level | Status |
|------|----------|------|----------|-------|--------|
| Valid coupon applies the discount | AC-1 | Functional | Yes | e2e | ✅ covered — e2e/checkout.spec.ts:88 |
| Coupon at exactly the minimum cart total | AC-1 | Boundary | Yes | unit | New |
| Expired coupon is rejected at checkout | code: `Coupon#expired?` | Negative | Yes | integration | New |

#### Coupon at exactly the minimum cart total

- **Verifies:** AC-1 · **Priority:** P0 · **Automate:** Yes — unit (RSpec, `spec/models/coupon_spec.rb`)
- **Preconditions:** A cart with subtotal exactly $10.00; an active, unused coupon.
- **Steps:**
  1. Apply the coupon to a cart with subtotal $9.99.
  2. Apply the coupon to a cart with subtotal $10.00.
- **Expected:** $9.99 is rejected with "Cart total must be at least $10 to use a coupon"; $10.00 accepts it.
- **Test data:** Coupon `SAVE10`, cart subtotals `$9.99` and `$10.00`.

### P1 — Should ship with it

<same shape>

### P2 — Nice to have

<same shape>

### Acceptance criteria with no case

- **AC-3** "Expired coupons are rejected" — expiry isn't defined precisely enough to pick
  a boundary; see Open questions below.

### ⚪ Not verifiable without more input

- 4 UI-state cases (empty cart with an applied coupon, coupon-field error style) need the
  design. Share a Figma link or a screenshot to complete them.

### Open questions for the PO

1. AC-3 doesn't define expiry — calendar day in the user's timezone, or a UTC instant?
   The boundary case at "1 second past expiry" depends on which.
```

Rules for each section:

- **Header line** (`**Source:** … **Design:** … **Code scanned:** …`) always states what
  was actually available — `**Design:** none available` is a valid, expected value, not an
  omission.
- **Coverage line** — the P0/P1/P2 counts and the covered count are the report's one-line
  summary; always present, even at "0 P0, 0 P1, 0 P2" for a feature with only ambiguities.
- **Priority sections** — omit a priority section entirely if it has zero cases and
  `testCases.scope`/`maxCases` didn't filter it out; don't print an empty table with
  `_(none)_` the way `pr-review`'s severity sections do — a QA plan with no P2s is normal,
  not a gap to call out.
- **`### Acceptance criteria with no case`** — only present when at least one AC has zero
  cases; omit entirely otherwise.
- **`### ⚪ Not verifiable without more input`** — only present when at least one visual/UX
  case was withheld for lack of a design source; state what's missing and how to provide it.
- **`### Open questions for the PO`** — only present when Lens 1/Ambiguities produced at
  least one; numbered, each naming the specific AC or case it blocks.
- Any config-driven skip — a disabled lens, `crossReferenceExistingTests: false`,
  `maxCases` truncation, `scope` filtering out a side — gets its own one-line disclosure at
  the bottom, same "never silent" rule as `pr-review`'s `ignorePaths`:
  ```markdown
  _Lens 8 (cross-cutting UX) disabled by config._
  _12 P2 cases omitted — maxCases: 20._
  ```

## Case Formats (`testCases.format`)

The per-case body above is the `steps` format (default). Two others, chosen per `testCases.format`
in config — same case, three renderings:

**`gherkin`** — `Given/When/Then`, with `Scenario Outline` + `Examples` for a boundary set
sharing one shape:

```gherkin
Scenario Outline: Coupon accepted only at or above the minimum cart total
  Given a cart with subtotal <subtotal>
  And an active, unused coupon
  When the coupon is applied
  Then the result is <result>

  Examples:
    | subtotal | result   |
    | 9.99     | rejected |
    | 10.00    | accepted |
```

**`table`** — one row per case, for pasting into a spreadsheet or an external tool that
doesn't read markdown case bodies:

```markdown
| Case | Verifies | Priority | Preconditions | Steps | Expected | Automate | Level | Status |
|------|----------|----------|----------------|-------|----------|----------|-------|--------|
| Coupon at exactly the minimum cart total | AC-1 | P0 | Active coupon, cart $10.00 | 1. Apply at $9.99 2. Apply at $10.00 | $9.99 rejected, $10.00 accepted | Yes | unit | New |
```

## Machine-Parseable Block (`### TEST_CASES`)

Optional, for a script or a follow-up automation task to consume the plan mechanically —
same sentinel pattern as `pr-review`'s `### FINDINGS`. Never printed unless the run is
explicitly for that purpose (config or an explicit request) — it's noise for a human
reading the plan directly.

```text
### TEST_CASES
- slug: coupon-at-minimum-cart-total
  title: Coupon at exactly the minimum cart total
  verifies: AC-1
  lens: boundary
  priority: P0
  automate: true
  level: unit
  framework: RSpec
  status: new
  steps: Apply coupon at $9.99; apply coupon at $10.00
  expected: $9.99 rejected, $10.00 accepted
### END TEST_CASES
```

`slug` is derived from the title (kebab-case, ASCII) — **not** a sequential counter; see
`docs/decision-records/2026-09-07--test-cases-carry-no-synthetic-ids.md`. If two cases
title-collide, disambiguate the slug with the lens (`-boundary`, `-negative`), never with a
running number. If there is nothing to report, return the literal empty block:

```text
### TEST_CASES
### END TEST_CASES
```

## Saving to a File (`output.file`)

Off by default. When `output.file` is set (e.g. `docs/qa/test-cases`), save the identical
markdown report — not a reformatted or summarized version — to
`<output.file>/<feature-slug>.md`, creating the directory if missing. The chat report is
still printed in full first; the file is a copy of the deliverable, never a replacement for
it. See `assets/test-cases-report.md` for the template.
