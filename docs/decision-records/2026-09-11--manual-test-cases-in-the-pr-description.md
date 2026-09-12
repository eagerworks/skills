# Add tickable manual test cases to `create-pr`'s `Test plan` section

- **Date:** 2026-09-11

## Context

`create-pr`'s only "how is this verified" section, `Test plan`, is author-side and automation-flavored: "reproducible steps with the exact commands and the observed result" (e.g. `npm test — 88/88 passed`). Nothing in the description this skill writes tells the dev or QA person who has to sign off on the PR's quality what to actually exercise by hand — what to click, which account to use, what the correct outcome looks like. That gap was concrete rather than hypothetical: grep across `skills/create-pr/` for "QA", "manual test", or "test scenario" returns nothing.

Four judgment calls had to be made, each with a real failure mode on the other side:

1. **Where does this live?** A brand-new `## Manual Test Cases` heading is the most visible option, but it would compete with `## Test plan` for the same reader question ("how do I know this works?"), and `pr.sections`, which already lists the six default headings, would need every downstream file's enumeration touched. Folding it into the existing `## Checklist` was also considered — the checklist already has boxes — but that mixes repo-wide merge gates ("no secrets in the diff") with QA instructions for this one PR, and a checklist item is meant to be answered once, not walked through step by step.
2. **On by default, or opt-in?** An opt-in design (`pr.manualTestCases.enabled: false` → `true`) means most repos, which never touch `.eagerworks/create-pr.json`, never get the benefit — the same problem `pr-review`'s documentation lens explicitly avoided by defaulting to on (`2026-08-21--documentation-decision-capture-lens.md`).
3. **What counts as "observable"?** Reusing the screenshot section's UI-visible gate outright (`pr.screenshots.uiPaths`) was the cheapest option, but it would silently exclude an API contract change, a new webhook, or a CLI flag — all things a QA person tests by hand without ever opening a browser.
4. **What happens with nothing to test?** Omitting the subsection when nothing qualifies mirrors `pr-review`'s `### Documentation` precedent, but a missing subsection is indistinguishable from a subsection nobody thought to write — exactly the ambiguity `2026-08-21--ignore-paths-must-be-disclosed.md` already ruled out for `ignorePaths`.

The failure mode with the highest cost here is specific to this feature: a fabricated manual test step is worse than a missing one. A wrong `TODO`-free step gets *followed* by a human, and a checkbox next to a scenario the diff doesn't actually exercise reads as false assurance to whoever is deciding the PR is safe to merge.

## Decision

1. **`## Test plan` gains two subsections, `### Automated` (unchanged) and `### Manual`**, rather than a new top-level heading or a checklist item. This keeps one answer to "how was this verified," split by audience, and leaves `pr.sections`'s default list (`Summary`, `Problem`, `Solution`, `Screenshots`/`Demo`, `Test plan`, `Checklist`) untouched — a repo that already pinned `pr.sections` still gets the new subsection with no config change, and `docs/decision-records/2026-09-07--repo-pr-template-resolved-by-asking.md` needs no revisiting, since it only calls for that when `pr.sections`' shape changes.
2. **On by default**, via `pr.manualTestCases.enabled` (default `true`) in `.eagerworks/create-pr.json`, mirroring the documentation lens's precedent — a repo gets the benefit without touching config, and can turn it off explicitly.
3. **Applicability is "any observable behavior," not just UI**: an HTTP endpoint, a job/worker/cron task, a CLI command, an email/notification/webhook, or a migration with a visible effect, in addition to a UI path — see `skills/create-pr/references/manual-test-cases.md`. Explicitly excluded, mirroring `pr-review` Lens 5's anti-nag list: behavior-preserving refactors, dependency bumps, CI/tooling config, docs-only, and test-only changes.
4. **Nothing to test is a one-line disclosure, not an omitted subsection** — `### Manual` followed by `_None — <reason>_` — so a reviewer can tell the section was evaluated and correctly came up empty, not skipped. The same disclosure rule applies when `pr.manualTestCases.enabled` is `false` or a repo-configured `maxScenarios` cuts the list.
5. **Every scenario is `#### N. <actor + outcome>` → one-line `**Setup:**` → `- [ ]` steps → a final `- [ ] **Expected:** …` step**, and every checkbox ships unticked — `create-pr` writes scenarios, it doesn't run them. Each step must name the exact route/screen/endpoint/command, the actor and account state, and concrete data, so it's executable by someone who hasn't read the diff. Anything the skill can't source — a staging URL, a test account, a feature-flag name — becomes a `TODO(author): <what's missing>` line, the exact placeholder convention `2026-09-07--screenshots-via-gh-attach-only.md` established for an unavailable screenshot, never a guess.
6. **No padding, and no cap by default.** `maxScenarios` is unset out of the box — the skill writes as many real scenarios as the diff supports, however many that is; two real scenarios beat five padded ones, and zero is valid when nothing qualifies. A repo that wants a ceiling opts in via `pr.manualTestCases.maxScenarios`; over that cap, keep the highest-risk scenarios (security/permissions, then data integrity, then regression-prone paths) and disclose the cut in one line.

## Consequences

- A PR now ships with concrete, tickable QA instructions by default, without the user asking for the feature by name — at the cost of one more subsection every description-writing run has to evaluate, even when the answer is "none."
- A repo that already pinned `pr.sections` for the PR-template question gets the new subsection automatically, since `Test plan` is still a single heading in that list.
- The fabrication ban is stricter here than for the checklist: a checklist item can be marked `N/A, <reason>` and stay useful; a manual test step has no equivalent safe degradation — an unsourceable detail must become a visible `TODO(author):`, never a filled-in guess, because a human will follow it.
- `references/manual-test-cases.md` is the new single source of truth; `SKILL.md`, `references/description.md`, `references/config.md`, both files in `assets/`, and `README.md` all point to it rather than duplicating its rules, per this collection's progressive-disclosure convention.

## Related

- [2026-09-07--screenshots-via-gh-attach-only](2026-09-07--screenshots-via-gh-attach-only.md) — the `TODO(author):`-placeholder precedent this record reuses for an unsourceable manual-test detail.
- [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) — the disclosure-never-silent rule applied here to `pr.manualTestCases.enabled: false` and to the `maxScenarios` cap.
- [2026-08-21--documentation-decision-capture-lens](2026-08-21--documentation-decision-capture-lens.md) — the on-by-default-with-a-cap design this record mirrors, including the "omit when empty" pattern this record deliberately departs from (a one-line disclosure instead).
- [2026-09-07--repo-pr-template-resolved-by-asking](2026-09-07--repo-pr-template-resolved-by-asking.md) — confirms this change needs no update there, since `pr.sections`' shape is unchanged.
- `skills/create-pr/references/manual-test-cases.md` — the operative procedure, scenario shape, and step-writing rules this record justifies.
- `skills/create-pr/references/config.md` — the `pr.manualTestCases` schema entry.
