# Loop recommendations in loop-engineering-audit are advisory and never enter the Work Plan or the verdict

- **Date:** 2026-09-12

## Context

The audit answers one question with a mechanical contract: graded checks → a Work Plan whose rows map 1:1 to 🔴/🟡 checks → a verdict computed from the grades alone. That 1:1 mapping is the whole anti-padding defense (`SKILL.md` gotcha "zero blockers is a valid result... never pad the Work Plan").

The skill was asked to also answer four new questions: what in the workflow could be automated, which agent loops the project could create *and maintain*, what risk each carries, and what to configure so each can run. That output is prescriptive and partly speculative — it has no check ids, no grade, and no natural cap. Bolting it onto the existing contract puts pressure on four places, each with a real failure mode on the far side:

1. **Where it lives in the report contract.** Recommendations in the Work Plan either break the 1:1 mapping or force inventing pseudo-checks to justify rows — and either way the verdict starts depending on a team's automation ambition instead of its readiness. A repo with a perfect 🟢 scorecard and no loops is *ready*; saying otherwise makes the verdict mean something new.
2. **Whether the skill scaffolds the loops.** "What to configure" is a short step from "here, I wrote it." But the skill has exactly one write by decision (`2026-08-28--audit-report-saved-to-docs.md`), and a CI workflow holding agent credentials plus a permissions file is the highest-consequence file this skill could author — generated from an audit the user hasn't read yet.
3. **How loops are classified by risk.** The obvious axis — "how likely is the agent to get this wrong" — is unmeasurable from a repo and biases optimistic. It also produces the wrong answer: a careful agent on a deploy loop is still a deploy loop.
4. **Padding.** A ten-entry catalog is an invitation to recommend ten entries, and a loop recommendation has no 🔴 to trace back to. Left unbounded, the section becomes the report's longest and least evidenced part.

There is also a softer risk worth naming: implying everything should be automated. A workflow map with only "automated / not yet automated" quietly asserts that production merge authorization and exploratory QA are gaps to close.

## Decision

1. `references/loop-catalog.md` is the single source of truth for this concern — the automation map, the risk ladder, the catalog, the selection rules. `SKILL.md` summarizes it in one section. **`references/rubric.md` gains no check and no grade from it** — the catalog only cites existing check ids (`1.6`, `3.5`, `8.3`) as loop prerequisites, and the rubric's Conservatism rule now says so explicitly, so `references/rubric.md` stays the single source of truth for grading.
2. Two new report sections — **4. Automation map** and **5. Recommended loops** — slot between the Work Plan and Findings by dimension (existing sections renumber to 6/7/8). Both carry a required, verbatim "advisory" subtitle. They add no Work Plan rows, change no grade, and never move the verdict or the Blockers/Gaps/Unverifiable counts. The header block is untouched — anything in it reads as part of the verdict. **The single permitted coupling** is a `Blocked by` column in which a loop cites the Work Plan item numbers that unblock it — section 5 points at section 3, never the reverse. This resolves judgment call 1, and mirrors `pr-review`'s Lens 5B precedent (`2026-08-21--documentation-decision-capture-lens.md`): a capped, severity-free, non-blocking suggestion list in its own section, never summed into the verdict.
3. The automation map's statuses are five plain words — **Automated / Assisted / Manual / Human by design / Absent** — deliberately not emoji and not a fifth grade, so they can never be visually summed into the verdict. **Human by design** exists specifically so the map can assert a stage should stay with a person; production merge authorization, deploy sign-off, exploratory QA, credential rotation and product prioritization are named as such in the catalog, and no loop may propose removing them. **Absent** exists so "there is no release process here" is a classification instead of a gap.
4. The risk ladder is **L1 Contained / L2 Shared-state / L3 External-effect**, defined by blast radius and reversibility — properties of the configuration, which a repo can prove — explicitly not by agent fallibility. This resolves judgment call 3. L3 is recommended only when an automated deploy, a tested rollback, and a readable post-deploy signal already exist; `loops.maxRiskLevel` defaults to `2` so a production-reaching loop is never proposed by default.
5. Judgment call 4 is resolved with four bounds: a cap of five (`loops.maxRecommended`), a deterministic four-key ordering (prerequisites met → risk → evidence strength → configuration effort), mandatory evidence per recommendation under the rubric's own conservatism rule, and an explicit never-recommend list (no deploy loop without a pipeline, no release-notes loop without releases, no quarantine loop without flake evidence, no dependency loop without a lockfile, nothing that duplicates an already-Automated stage). Zero recommendations is a valid and expected result, rendered as a one-line "the first loop you unlock is X, once Work Plan items #N–#M are done" — for a repo with 🔴s that *is* the answer to "which loops should we run". Every candidate removed by a cap or the risk filter is disclosed by count in the footer, reusing the disclosure discipline of `2026-08-21--ignore-paths-must-be-disclosed.md`.
6. Judgment call 2: **the skill still writes exactly one file.** Playbooks, workflows, permission files and labels are described under each loop's "what to configure" and left for the team to build; no template ships with the skill. The new workflow phase (Phase 5, between the Work Plan and delivery) **executes nothing**: the map is built from source plus the read-only `gh` GET probes that already live in Phase 2, which remains the only execution step. Three read-only probes (`gh release list`, `.../environments`, `.../actions/workflows`) were appended there rather than to the new phase for exactly that reason.
7. Maintenance is part of the recommendation, not an afterthought: every recommended loop names an owner, a review cadence, a kill switch, and an exit condition directly in the report block. A loop nobody owns was treated as a loop that shouldn't be recommended.

Rejected: folding loops into the Work Plan as low-priority items; a `loops.allow`/`deny` name list (it would become this skill's one silent skip); generating the workflow and permissions files; a per-loop "confidence" score in place of the risk ladder; and shipping a copyable playbook template asset — the "what to configure" and maintenance guidance live in the report and in `references/loop-catalog.md` only, so the skill's asset surface doesn't grow for this feature.

## Consequences

- The report grows two sections and the audit does more reasoning per run. Bounded by the cap of five and by the "none" case collapsing to a single sentence.
- `loops.enabled: false` exists for teams that run the audit purely as a readiness gate; it's disclosed in the footer like a disabled dimension, never a silent omission.
- Delivery is now **Phase 6** of `references/audit-workflow.md`. The 2026-08-28 record's Related pointer was changed from "Phase 5, delivery" to "the delivery phase" — the number was dropped rather than renumbered, so a future phase insertion can't re-stale it.
- A future reviewer asked to "make the audit set up the loops too" should read this record first: the answer is the "what to configure" field describing the change, not a second write. The one-write line from 2026-08-28 is deliberately reaffirmed under real pressure here.
- The catalog is now a maintained list. A new loop gets added with all nine fields, a risk level, and a "recommend when" tied to repo evidence — or it doesn't get added.
- The skill can now tell a team that a stage should *stay* manual, which is new and occasionally unwelcome output for a skill whose name invites automation-first expectations.
- The catalog's concurrency floor (running loops in parallel) depends on dimension 8 from the sibling record; when it's unmet, recommended loops still appear but are marked to run sequentially rather than filtered out.

## Related

- [`2026-08-28--audit-report-saved-to-docs.md`](2026-08-28--audit-report-saved-to-docs.md) — the one-write rule this decision reaffirms rather than relaxes.
- [`2026-08-21--documentation-decision-capture-lens.md`](2026-08-21--documentation-decision-capture-lens.md) — the capped, severity-free, non-blocking suggestion-section precedent in `pr-review` that section 5 follows.
- [`2026-08-21--ignore-paths-must-be-disclosed.md`](2026-08-21--ignore-paths-must-be-disclosed.md) — the disclosure precedent reused for `loops.enabled: false` and for the cap/risk-filter counts.
- [`2026-09-12--parallel-session-readiness-is-a-capped-dimension.md`](2026-09-12--parallel-session-readiness-is-a-capped-dimension.md) — the sibling record; its dimension 8 backs this catalog's concurrency floor.
- `skills/loop-engineering-audit/references/loop-catalog.md` — the automation map, risk ladder, catalog and selection rules this record justifies.
- `skills/loop-engineering-audit/references/output-format.md` — sections 4 and 5, and the Rules that keep them advisory.
- `skills/loop-engineering-audit/references/config.md` — the `loops` schema entry.
