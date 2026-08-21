# PR Review — Output Format

## Markdown Report (Default)

Use this format for a human reading the review in an IDE or terminal. One verdict line at the top, then findings grouped by severity, each with `file:line`, the claim, the concrete failure scenario, and a one-line fix.

```markdown
## PR Review

**Verdict:** 1 critical, 1 high, 0 minor — do not merge until the critical finding is fixed.

### Critical

- **app/controllers/invoices_controller.rb:12** — `Invoice.find(params[:id])` is not scoped to the current user's organization.
  - **Failure:** an authenticated user from Org A can read any invoice by guessing or incrementing the id, regardless of which org it belongs to.
  - **Fix:** scope through the caller — `current_user.organization.invoices.find(params[:id])`.

### High

- **spec/requests/invoices_spec.rb** — acceptance criterion "a user cannot view another org's invoice" has no test.
  - **Failure:** nothing currently asserts the 404/403 behavior for a cross-org request; the scoping bug above could regress silently.
  - **Fix:** add a request spec that logs in as a user from Org B and asserts a 404 on Org A's invoice id.

### Minor

_(none)_

### Documentation

_(suggestions — not merge blockers, not counted in the verdict)_

- **Pessimistic lock in `SeatReservation#reserve!` instead of the repo's usual optimistic `lock_version`** — no comment or doc explains why; a future reader can't tell this was deliberate.
  - **Where:** `docs/decision-records/2026-08-21--seat-reservation-locking.md` — matching this repo's existing `YYYY-MM-DD--slug.md` convention.
  - **Rationale found:** the PR description says "optimistic retries were thrashing under the checkout-page burst" — worth capturing; it isn't in the repo anywhere.

_Skipped by ignorePaths: package-lock.json, src/generated/**_
```

Only include the `_Skipped by ignorePaths: ..._` line when `.eagerworks/pr-review.json` sets `review.ignorePaths` and at least one touched file matched — see `references/config.md`. Omit it entirely otherwise; never print it empty.

Only include the `### Documentation` section when there's at least one suggestion — unlike the severity sections, **do not** print `_(none)_` for an empty documentation section; the verdict line already proves the review ran, and a section that's always present even when empty is exactly the noise this lens is designed to avoid. The `**Verdict:**` line itself only counts findings — a review with documentation suggestions and zero findings still reads e.g. "0 critical, 0 high, 0 minor".

When `.eagerworks/pr-review.json` sets `review.documentation.enabled: false` (`references/config.md`), Lens 5 doesn't run at all — no doc-drift findings, no suggestions — and the report says so with its own disclosure line, same principle as `ignorePaths`:

```markdown
_Documentation lens disabled by config._
```

When there is nothing to report, say so plainly instead of omitting the section:

```markdown
## PR Review

**Verdict:** 0 findings. The diff follows the repo's existing scoping and error-handling patterns; no correctness, security, or coverage gaps found.
```

## Machine-Parseable Block (Optional)

Use this when another skill, script, or agent needs to parse the output mechanically — for example, feeding findings into an automated fix loop. Not needed for a human reading the review directly.

```text
### FINDINGS
- id: F1
  severity: critical | high | minor
  file: path/to/file.rb:12
  claim: <one line — what is wrong>
  failure: <one line — concrete inputs/state producing the wrong outcome>
  fix: <one line — the concrete change>
### END FINDINGS
```

If there is nothing to report, return the literal empty block — never replace it with prose like "looks good" or "no issues found", since a caller parsing this block mechanically expects exactly one of the two shapes:

```text
### FINDINGS
### END FINDINGS
```

Doc drift (Lens 5A) is a normal finding and belongs in the `### FINDINGS` block above like any other. Documentation *suggestions* (Lens 5B) get their own parallel block, since they carry no severity and aren't findings:

```text
### DOCUMENTATION
- id: D1
  subject: <one line — the decision that was made>
  where: <proposed path, e.g. docs/decision-records/2026-08-21--seat-reservation-locking.md>
  why: <one line — the alternative it beat and why a future reader can't tell>
  rationale: <sourced quote from the PR/issue/commit/code, or the literal `not stated`>
### END DOCUMENTATION
```

If there are no suggestions, return the literal empty block, same rule as `### FINDINGS`:

```text
### DOCUMENTATION
### END DOCUMENTATION
```

## PR Comment

When posting to a PR (see `references/workflow.md`), use one line per finding, however long — never hard-wrapped; let GitHub wrap it. Order findings most-severe-first, and mark each with its severity:

- 🔴 `critical`
- 🟠 `high`
- 🟡 `minor`
- 📄 documentation suggestion (Lens 5B — always place these after every severity line)

```text
🔴 critical — app/controllers/invoices_controller.rb:12 — Invoice#show isn't scoped to the current user's organization, so any authenticated user can read any org's invoice by id. Fix: scope through `current_user.organization.invoices.find(params[:id])`.
🟠 high — spec/requests/invoices_spec.rb — acceptance criterion "a user cannot view another org's invoice" has no test. Fix: add a request spec asserting a 404 on another org's invoice id.
🟡 minor — app/services/reports/list_for_user.rb:8 — `call` could default `order` to a named scope for consistency with sibling services. Fix: extract `Report.recent` and use it here.
📄 Suggestion, not a blocker: app/models/seat_reservation.rb:8 — pessimistic lock chosen over the repo's usual optimistic `lock_version` with no recorded rationale. Consider `docs/decision-records/2026-08-21--seat-reservation-locking.md`.
```

If there is nothing to report, post the one-line verdict from the markdown report (e.g. "0 findings — no correctness, security, or coverage gaps found"), never an empty comment.
