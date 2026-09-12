# `review.ignorePaths` must always be disclosed in the report

- **Date:** 2026-08-21

## Context

`pr-review`'s workflow requires reading every touched file in full, not just the changed hunks (`SKILL.md`, `references/workflow.md`) — the diff alone hides the surrounding context lenses 1–3 need. On a diff that carries a `package-lock.json`, a generated Prisma/GraphQL client, or a snapshot file, that rule burns context on files that were never hand-written and produces noise findings (style nits on generated code) that add nothing.

A repo-configurable skip is the obvious fix, but a code review tool skipping paths is a deliberate blind spot, and blind spots in a review tool are exactly the kind of thing that erodes trust if they're invisible. The skill already has a precedent for this tension: `references/workflow.md` → "Reviewing a Large Diff" requires saying explicitly which areas got a lighter pass on an oversized diff, specifically so "a partial review never reads as exhaustive." `ignorePaths` needed the same guarantee, not a weaker one, because unlike the large-diff case (the agent's own judgment call, visible in the same review), `ignorePaths` is repo-authored config the agent didn't choose — it's easier for it to become invisible over time as a `.eagerworks/pr-review.json` accretes patterns nobody revisits.

The other open question: what if a repo's own pattern accidentally (or carelessly) catches something that matters — a schema migration matched by an overly broad `db/**` entry, say? Lens 2 (security & data integrity) exists specifically to catch migration and scoping problems; silently trusting the config there would let a config typo suppress the review's own safety net.

## Decision

`review.ignorePaths` in `.eagerworks/pr-review.json` (`references/config.md`) takes glob patterns. Any touched file matching one is:

1. Excluded from the full-file read and produces no findings, **and**
2. Named in the report — a `_Skipped by ignorePaths: <patterns>_` line (`references/output-format.md`) — whenever a pattern actually matched something in this diff. No match, no line; never printed empty.

One exception, checked before the skip is applied: if a matched file is a schema migration or auth/scoping code — the exact class of file Lens 2 exists to catch — it is read and reviewed anyway, and the report calls out that the configured pattern would have excluded it. A repo's own config can create a blind spot for generated noise; it cannot silently suppress the review's core safety net.

Silent, undisclosed skipping was considered and rejected: it's the simplest option but directly contradicts the disclosure principle already established for large-diff triage, and a review that quietly omits a whole area is worse than one that flags the area as light-touch.

## Consequences

- A review against a repo with `ignorePaths` configured is measurably faster and quieter on large diffs that touch lockfiles or generated code, at the cost of one extra line in the report when a pattern actually matches.
- A repo that writes an overly broad pattern (e.g. `db/**`) doesn't get a silent free pass on migrations — the review reads and reports on them regardless, and flags the pattern as too broad.
- `assets/pr-review.example.json` and both examples in `references/config.md` model `ignorePaths` with realistic Rails and Node entries (`db/schema.rb`, `package-lock.json`, `src/generated/**`), so a repo copying the starter sees the intended scope.

## Related

- [2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md) — the other precedent in this skill for "ask/disclose rather than silently guess."
- `skills/pr-review/references/config.md` — the `ignorePaths` schema entry this ADR justifies.
- `skills/pr-review/references/workflow.md` → "Reviewing a Large Diff" — the pre-existing disclosure principle this decision extends to repo-configured skips.
