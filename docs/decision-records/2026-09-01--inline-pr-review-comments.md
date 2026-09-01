# 8. Post `pr-review` findings as an inline GitHub review, not just a summary comment

- **Date:** 2026-09-01

## Context

Since [2026-08-21--optional-fix-loop-and-round-cap](2026-08-21--optional-fix-loop-and-round-cap.md) introduced posting to a PR, the skill's only mutation there was `gh pr comment <N> --body-file <report>` — a single issue comment on the Conversation tab, unattached to the diff. Every finding already carries a `file:line` (`references/output-format.md`), but a reader has to jump from the comment back into the Files Changed tab and find the line by hand. GitHub's own review UI — inline comments anchored to the exact line — exists precisely to remove that jump, and it's the format most reviewers already expect from an automated review.

Three questions needed a settled answer before touching `references/workflow.md`:

1. `gh pr review` (the CLI's dedicated review command) turns out not to support this at all — verified against the installed `gh` 2.83.2, its whole flag set is `--approve/--comment/--request-changes/--body/--body-file/--repo`, no `--path`/`--line`. Only `gh api` against `POST /repos/{owner}/{repo}/pulls/{n}/reviews` can create inline comments, batched with a review.
2. That endpoint is **atomic**: every comment in the `comments[]` array must anchor to a line that is actually part of the PR's diff (an added, deleted, or context line inside a hunk), or the whole POST 422s and *nothing* posts — including findings that would have anchored fine. But not every finding has an anchor: a missing-test finding often names a whole file with no line, and a Lens 5B documentation suggestion isn't about a line at all. Silently dropping those to keep the review atomic would make the inline mode strictly worse than the summary comment it replaces.
3. A GitHub review requires an `event`: `APPROVE`, `REQUEST_CHANGES`, or `COMMENT`. `APPROVE` 422s outright on a PR the caller authored, and `REQUEST_CHANGES` makes the skill start gating merges — a policy the skill has never claimed and shouldn't adopt implicitly as a side effect of an output-formatting change.

## Decision

1. **Compute anchorability from the diff before building any payload.** `gh api .../pulls/<N>/files` gives each file's `patch`; walk it to build the set of commentable `path:line` pairs on `side: RIGHT` (v1 scope: single lines only, no `start_line` ranges, no `LEFT` side). A finding is anchorable only if it names a `file:line` in that set.
2. **Split, never drop.** Anchorable findings become inline comments, posted on their exact line. Every unanchorable finding — no line, an out-of-hunk line, a `patch: null` (binary/generated) file, every documentation suggestion — is kept **in full** in the review's summary body, under its normal severity heading. The summary body also carries the verdict line and the existing disclosure lines (`ignorePaths`, documentation-lens-disabled). Nothing the console report shows is ever lost by going inline.
3. **`event` is always `"COMMENT"`.** Never `APPROVE`, never `REQUEST_CHANGES` — consistent with the skill's read-only, advisory posture; the verdict line alone communicates mergeability.
4. **Degrade, don't fail.** If the review POST 422s, re-fetch the head SHA (a push between diffing and posting moves every line), retry once if it changed. If it still fails, fall back to exactly today's behavior — `gh pr comment <N> --body-file <report>` with the full report — and disclose the fallback in one line. The console report is always printed first regardless of what happens next, same rule as a missing/unauthenticated `gh`.
5. **New `review.commentStyle: "inline" | "summary"`** in `.eagerworks/pr-review.json`, default `"inline"`. `postToPr` stays the pure on/off switch; `commentStyle` only picks the shape once posting is already happening. `"summary"` reproduces the pre-existing single-comment behavior verbatim — for a repo that prefers one comment, or wants identical output across a mixed fleet of tools where inline reviews render inconsistently.

The batched `reviews` endpoint was chosen over the per-comment `POST /pulls/{n}/comments` endpoint despite the atomicity risk above: per-comment posting isolates failures but costs one API call and one notification per finding (against a roughly 80/min secondary rate limit), produces no single review event a reader can see at a glance, and still needs the same anchorability check to avoid 422s one comment at a time instead of once.

## Consequences

- A reviewer reading a PR now sees findings on the lines they describe, the same way a human reviewer would leave them — with the verdict and anything unanchorable still in one place at the top.
- Repos that pinned nothing in `.eagerworks/pr-review.json` get inline reviews by default; a repo that explicitly wants the old behavior sets `commentStyle: "summary"` once.
- The atomic-POST risk is fully absorbed by the anchorability preflight and the fallback ladder — a review can degrade to a summary comment, but it can never post nothing and never silently drop a finding.
- `assets/code-reviewer.agent.md`'s rule that the subagent never posts is unaffected — posting, in either shape, stays the caller's job.

## Related

- [2026-08-21--optional-fix-loop-and-round-cap](2026-08-21--optional-fix-loop-and-round-cap.md) — introduced posting to a PR at all; this record only changes its shape.
- [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) — the precedent this record follows: degrade or skip visibly, never silently.
- `skills/pr-review/references/workflow.md` → "Posting to a PR" — the operative procedure this ADR justifies.
- `skills/pr-review/references/output-format.md` → "Inline Review Comments" — the exact body split between inline comments and the summary.
