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

_Skipped by ignorePaths: package-lock.json, src/generated/**_
```

Only include the `_Skipped by ignorePaths: ..._` line when `.eagerworks/pr-review.json` sets `review.ignorePaths` and at least one touched file matched — see `references/config.md`. Omit it entirely otherwise; never print it empty.

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

## PR Comment

When posting to a PR (see `references/workflow.md`), use one line per finding, however long — never hard-wrapped; let GitHub wrap it:

```text
🔴 critical — app/controllers/invoices_controller.rb:12 — Invoice#show isn't scoped to the current user's organization, so any authenticated user can read any org's invoice by id. Fix: scope through `current_user.organization.invoices.find(params[:id])`.
```
