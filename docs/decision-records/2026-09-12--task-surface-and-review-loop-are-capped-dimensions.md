# Task surface and the review & release loop are capped dimensions in `audit-open-source-repo`

- **Date:** 2026-09-12

## Context

`loop-engineering-audit` established the precedent that a dimension can be graded on the same four-grade scale as every other dimension while never rolling up to 🔴 — `2026-09-12--parallel-session-readiness-is-a-capped-dimension.md` caps parallel-session readiness because a repo that can't run several agent sessions at once can still run one; the gap caps throughput, not the possibility of using the skill at all.

`audit-open-source-repo` has two dimensions with the same shape, for two different underlying reasons, and both needed the same treatment before the rubric was written rather than discovered as an inconsistency after the fact.

**Dimension 5, Task surface**, asks whether there's something specific, scoped, and unclaimed for a newcomer to pick up. A repo with zero issues labeled `good first issue`, no roadmap, and poor issue triage is genuinely worse for a newcomer who wants to be handed work — but it is not closed to a contributor who already knows what they want to build and shows up with their own feature idea. Grading an empty task surface 🔴 would conflate "nobody curated an easy on-ramp" with "the door is shut," and would make every quiet, small, single-maintainer project that happens to work fine without ever labeling an issue report as **Closed to contributors**, which is simply false.

**Dimension 10, Review & release loop**, is architecturally different from every other dimension in this rubric: it is the only one graded from **people's behavior over a sample of history** (median time to first review, external-PR merge rate, open-PR staleness) rather than from an artifact that exists or doesn't. A median computed over a dozen PRs is noisy, is sensitive to the specific historical window sampled, and — most importantly — is a judgment about a team's responsiveness, not about whether their project's structure permits a good contribution. Blocking the verdict on it risks the single failure mode most likely to make a maintainer discard the whole report as unfair: "you're telling me my project is *closed* because we're slow to review, when the actual mechanics of contributing here work fine?"

The risk with capping either dimension is that the cap becomes a way to hide a genuinely under-resourced or neglected project behind a 🟢 or 🟡 verdict that undersells how bad the situation is. `2026-09-12--parallel-session-readiness-is-a-capped-dimension.md` didn't need a compensating rule because a 🟡 there is not embarrassing in the way "we take 34 days to review a PR and 0 of 8 external contributions were ever merged" is — this domain needed one.

## Decision

1. **No check in dimension 5 (Task surface) or dimension 10 (Review & release loop) is ever graded 🔴.** Both dimensions cap themselves at 🟡 no matter how many checks fail or how severe the evidence — see `references/dimensions.md` → "Capped dimensions" for the two dimension-specific rationales above.
2. **Compensating rule, unique to dimension 10:** when any 10.x check is graded 🟡, the report's lead paragraph — normally three sentences summarizing the biggest break and win — is instead required to open with the stewardship numbers themselves (median time to first review, external PRs merged vs. opened, oldest unanswered PR), before any other summary sentence. The cap changes the verdict's ceiling; it never changes how prominently the underlying numbers are surfaced.
3. **Dimension 10's findings are always numbers, never characterizations.** "Median time to first review is 34 days across 8 external PRs" is a permitted finding; "the maintainers are unresponsive" is not — this is the same distinction `references/dimensions.md`'s conservatism rule and `SKILL.md`'s gotcha 5 draw between evidence and judgment about people.
4. **Both caps are load-bearing at the rubric-design stage, not applied as an exception afterward** — `references/dimensions.md` states them next to the grade ladder itself, and the exhaustive 🔴 list in the same file structurally excludes every 5.x and 10.x check by omission.

## Consequences

- A repo with a mediocre task surface or a slow review culture, but otherwise sound onboarding, correctly grades 🟡 Open with friction rather than 🔴 Closed to contributors — the verdict tracks whether the door is shut, not whether the welcome mat is optimal.
- The stewardship-numbers-first rule means a maintainer cannot get a soft, generic three-sentence summary while their PR backlog quietly sits at a 🟡 — the numbers are unavoidable in the one place every reader looks first.
- This collection now has a repeatable pattern for capped dimensions across two skills (`loop-engineering-audit`'s parallel-session readiness, this skill's task surface and review loop): a dimension whose worst case caps a resource or a courtesy, not the fundamental possibility of the activity the skill is auditing for, is a candidate for capping — but a capped dimension that could plausibly hide something embarrassing needs its own disclosure rule, not just the cap.
- Future dimensions added to this rubric that are graded from history rather than artifacts (a hypothetical "issue triage quality" dimension, say) should default to being capped and carrying a similar numbers-first rule, per this precedent, rather than being evaluated case by case.

## Related

- [2026-09-12--parallel-session-readiness-is-a-capped-dimension](2026-09-12--parallel-session-readiness-is-a-capped-dimension.md) — the original precedent for a dimension that never blocks the verdict, in `loop-engineering-audit`.
- [2026-09-12--loop-recommendations-are-advisory](2026-09-12--loop-recommendations-are-advisory.md) — a related but distinct guardrail in the sibling skill: advisory output that can't touch the verdict at all, versus this record's graded-but-capped dimensions that still do (up to 🟡).
- `skills/audit-open-source-repo/references/dimensions.md` → "Capped dimensions" and "Dimension 10" — the checklist entries and grading rule this ADR justifies.
- `skills/audit-open-source-repo/SKILL.md` → gotcha 5 — "never grade maintainer behaviour from a document," the companion rule to this record's point 3.
