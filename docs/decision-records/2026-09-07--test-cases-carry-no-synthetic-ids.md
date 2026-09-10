# `design-qa-test-cases` reports carry no synthetic case IDs

- **Date:** 2026-09-07

## Context

Every QA tool a case might end up in — Jira, Linear, TestRail, Xray, Zephyr — identifies its test cases with an ID, and `pr-review`'s own `### FINDINGS` block sets a precedent in this collection for numbering machine-readable items (`id: F1`, `id: D1`). The natural first draft of this skill's report format followed the same shape: `TC-01`, `TC-02`, … per case, referenced from a traceability matrix mapping each acceptance criterion to its IDs.

Two problems surface as soon as the ID is asked to do anything beyond decorate a row:

- **The skill has no registry, so an ID can't be stable.** There is no database or file tracking "TC-04 is permanently the coupon-boundary case." Every run reasons about the feature fresh. If a ticket's acceptance criteria change and the skill re-runs, cases are re-derived in whatever order the lenses produce them that time — `TC-04` silently becomes a different case than it was last run. An ID that looks stable but isn't is worse than no ID: a QA who wrote "see TC-04" in a bug report now points at the wrong case.
- **It collides with the real ID the destination assigns.** The moment a case is pushed to Jira, Linear, or TestRail (`references/integrations.md`), that tool mints its own key — `QA-482`, a TestRail case ID. A `TC-04` printed alongside it is a second, competing identifier for the same thing, and the two only ever coincide by accident.

The underlying need an ID was serving — referring to a case, and showing which cases cover which acceptance criterion — doesn't actually require a counter.

## Decision

1. The report mints **no sequential or synthetic case identifiers**. A case's identity is its **title**, which the skill already writes to be descriptive and unique within the feature's plan (`references/techniques.md`).
2. Traceability is a **field on the case, not a matrix keyed by ID**: every case carries a `Verifies:` value — the acceptance criterion it satisfies, or `code: <signal>` when it was derived from a validation/policy/enum with no matching AC. `### Acceptance criteria with no case` reports the gap directly, as a list of ACs, not as empty cells in an AC × case-ID grid (`references/techniques.md` → Traceability).
3. The `### TEST_CASES` machine-parseable block (`references/output-format.md`) uses a **title-derived slug** (kebab-case, ASCII) for callers that need a stable-within-this-run key, never a counter. A title collision is disambiguated by appending the lens (`-boundary`, `-negative`), never a running number.
4. `SKILL.md` states this as Critical Gotcha #5, since it's the kind of omission a future contributor would "fix" by reflex, having seen `pr-review`'s `id: F1` pattern.

## Consequences

- `pr-review`'s `id: F1`/`id: D1` convention is **not** a collection-wide precedent to copy by default — that ID is scoped to one review run and never leaves the PR it was posted to, so it doesn't face either problem this record describes. A future skill should decide IDs on their own merits, not by matching `pr-review`.
- A destination integration (`references/integrations.md`) is responsible for its own ID once a case is pushed — this skill's report never tries to predict or reserve one.
- Referencing a specific case in conversation ("the coupon-boundary case," "the third P0 case") works the same way a QA would talk about it before this skill existed — by description or position, not by a code the skill invented for the occasion.
- The eval suite must include a case that re-runs the skill on a feature whose ticket changed between runs, and checks the response doesn't reuse or imply stability for `TC-NN`-shaped identifiers it never had to begin with.

## Related

- [`skills/design-qa-test-cases/references/techniques.md`](../../skills/design-qa-test-cases/references/techniques.md) — the `Verifies:` field and the gap-list traceability section this record specifies.
- [`skills/design-qa-test-cases/references/output-format.md`](../../skills/design-qa-test-cases/references/output-format.md) — the title-derived slug in the `### TEST_CASES` block.
- [`skills/design-qa-test-cases/SKILL.md`](../../skills/design-qa-test-cases/SKILL.md) — Critical Gotcha #5.
