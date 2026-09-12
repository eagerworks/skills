# `audit-open-source-repo` never recommends adopting a CLA or DCO

- **Date:** 2026-09-12

## Context

Every other gap this audit finds has an obvious, low-cost fix the Contributor Readiness Plan can safely recommend: add a `CONTRIBUTING.md`, commit a linter config, split a secret out of a fork-triggered CI job. A missing Contributor License Agreement (CLA) or Developer Certificate of Origin (DCO) sign-off requirement looks, on the surface, like the same kind of gap — CONTRIBUTING.md doesn't mention one, so surely the Plan should suggest adding one.

It isn't the same kind of gap, for two reasons specific to this artifact:

1. **A CLA is a legal and organizational decision, not a technical one.** Adopting a CLA typically requires an entity (a foundation, a company, an individual maintainer acting as one) to hold the assigned or licensed rights, a process to collect signatures, and often a CLA-bot integration — none of which this audit can put in place, verify is legally sound, or reason about with the project's specific circumstances (whether it has a foundation home, whether maintainers want copyright assignment, whether the project's license already covers what a CLA would add). Recommending one without that context is recommending a decision the audit isn't positioned to make.
2. **A CLA has a real, well-documented contributor-deterrence cost.** Requiring a CLA measurably reduces the pool of people willing to open a first PR — it's exactly the kind of friction this entire skill exists to remove. A skill whose stated purpose is "make it easier for outside contributors to land quality features" recommending a control whose primary effect is "make it harder for outside contributors to submit anything at all" would contradict its own thesis, unprompted.

The tension: check 2.6 (contribution-agreement stance explicit and consistent) still has to exist, because a CLA/DCO check running silently in CI with no CONTRIBUTING mention is a genuine 🔴-worthy barrier — a contributor whose PR is blocked by a bot they were never told about is exactly the kind of first-contribution failure this audit is built to catch. The rubric needed a way to grade *consistency* between what's documented and what's enforced without ever grading *absence* as something to fix by adding an agreement.

## Decision

1. **The audit never recommends adopting a CLA or DCO**, regardless of project size, license, or any other signal. This is stated as `SKILL.md` gotcha 7 and repeated in `references/config.md`'s comment on `project.contributionAgreement`, so it survives independent of any single reference file.
2. **Check 2.6 grades consistency only, never presence:**
   - A CLA/DCO check running on PRs with no document mentioning it → 🔴 (the barrier is the silent surprise, not the agreement itself).
   - A documented sign-off requirement with nothing enforcing it → 🟡 (contributors can't tell if they actually need to comply).
   - Neither present → 🟢. **"No agreement required" is stated as a complete, common, and usually correct answer** — not a lesser-graded fallback state.
3. **`project.contributionAgreement`** (`"none"` | `"dco"` | `"cla"`) records the project's own declared stance for the audience-aware n/a logic elsewhere in the rubric, but is explicitly documented as input the maintainers provide about a decision they've already made — never a lever this audit uses to suggest they make a different one.
4. If a user directly asks whether they should add a CLA (as opposed to asking for the readiness audit), the skill states its position plainly — that it's a legal/political choice with a real contributor-deterrence cost, outside this audit's scope — rather than silently declining to answer or, worse, quietly producing a recommendation anyway because the question was asked directly.

## Consequences

- A well-run project with no CLA and nothing requiring one is graded 🟢 on check 2.6 without qualification — the audit doesn't imply that adding one would somehow be *more* correct.
- The Contributor Readiness Plan can never contain a row that says "add a CLA," "add a DCO bot," or equivalent, under any evidence — this is a hard exclusion enforced by the check's grading rule itself (point 2 above), not by an editorial pass over the Plan afterward.
- A project that *does* want a CLA gets no obstruction from this audit either: it still passes 2.6 as long as the requirement is documented and consistently enforced. The audit takes no position on whether the choice was wise — only on whether it's been communicated honestly to the people it affects.
- This is the collection's first explicit "never recommend X" carve-out inside a Plan/Work-Plan-shaped skill; `loop-engineering-audit`'s equivalent guardrails (never recommend a loop the repo can't run) are about feasibility, not about the audit declining to take a substantive position on a legal/political question. Future skills that touch legal or governance-adjacent territory (a security or licensing audit, say) should look at this record before assuming "found a gap → recommend closing it" is always the right shape for that class of finding.

## Related

- `skills/audit-open-source-repo/SKILL.md` → gotcha 7 — the operative rule this ADR justifies.
- `skills/audit-open-source-repo/references/dimensions.md` → Dimension 2, check 2.6 — the grading table this ADR's point 2 describes.
- `skills/audit-open-source-repo/references/config.md` → `project.contributionAgreement` — the config field that records the project's stance without ever driving a recommendation.
- [2026-09-12--task-surface-and-review-loop-are-capped-dimensions](2026-09-12--task-surface-and-review-loop-are-capped-dimensions.md) — a sibling guardrail in the same skill that also exists to prevent a technically-accurate finding from being reported in a way that misleads or unfairly indicts a project.
