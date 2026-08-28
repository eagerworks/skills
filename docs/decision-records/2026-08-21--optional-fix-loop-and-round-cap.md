# 5. Gate the fix loop behind explicit opt-in, and cap it with three independent stop conditions

- **Date:** 2026-08-21

## Context

`pr-review`'s default behavior is read-only: it reports findings and touches nothing (`SKILL.md` → Critical Gotchas #1). That default was never in question. What needed deciding was the shape of the *opt-in* alternative — a user can ask for findings to be fixed, and the skill then has to review, edit, re-verify, and re-review in a loop until the diff is clean. Four judgment calls had a real failure mode on both sides:

1. **Should fixing ever happen without being asked?** A skill that edits files on its own initiative — even to fix something it just found wrong — contradicts the read-only-by-default rule the rest of the skill is built on (`SKILL.md` #1), and a surprise edit is a worse outcome than a report the user has to act on themselves.
2. **When does the loop stop on its own vs. hand control back to a human?** An uncapped review-fix-review loop can thrash: a fix for one finding can introduce or reveal another, and nothing guarantees convergence. But a hard round limit that fires regardless of what's happening cuts off a loop that's making real progress just as easily as one that's stuck.
3. **Does "no critical/high findings left" mean the loop is actually done, or just that this round happened to be clean?** A single clean round is weak evidence — the same bug can resurface after a later-round fix perturbs the code back into a bad state. Dogfooding this skill's own PR (`0e67944`) found this exact ambiguity: the original wording didn't say what happens to `minor` findings surviving the stopping round, and a plausible reading was "keep looping until literally zero findings of any severity," which would chase style nits forever instead of closing the gaps the loop exists for.
4. **What counts toward the round cap?** Once Lens 5 (`2026-08-21--documentation-decision-capture-lens.md`) added documentation *suggestions* — never a severity-rated finding, always sourced or explicitly marked `TODO(author):` — those had to be kept out of the same counter as `critical`/`high` findings. A suggestion isn't a defect the loop is closing; counting it as one would either stall the loop on an artifact it can't "fix" by editing code, or silently launder an unsourced guess into the repo's permanent decision log just to make the round cap happy.

## Decision

**Opt-in only** (judgment call 1). The fix loop (`references/workflow.md` → "Optional Fix Loop") runs only when the user explicitly asks for findings to be fixed; the standard review never invokes it.

**The loop, once opted into:**
1. Review — produce findings via the five lenses.
2. Fix — implement a change for every `critical`/`high`/`minor` finding, or decline one only by citing a specific documented convention the code follows, or by demonstrating the claimed failure path can't occur, recorded in one sentence. A Lens 5A doc-drift finding is fixed the same way (edit the stale doc); a Lens 5B suggestion is drafted as a proposed file, sourced only from the PR/issue/commits/diff, with anything unsourceable left as an explicit `TODO(author):` line rather than a guess — and this drafting step is explicitly exempted from the round cap and the identical-findings check (judgment call 4), since it produces no severity-rated finding to close.
3. Re-verify — run the repo's own `localChecks` (tests, linter, type checker) against the new changes.
4. Re-review — review only what changed this round, against the same five lenses.
5. Repeat or stop: any `critical`/`high` finding this round sends it back to step 1; a `minor`-only or zero-finding round ends the loop, with any surviving `minor` findings reported as known remaining nits rather than chased further (judgment call 3 — this line is the direct fix from `0e67944`, making explicit that the loop's job is closing critical/high gaps, not zeroing every style nit).

**Three independent stop conditions** (judgment call 2), any one of which ends the loop and hands control back to the user rather than continuing indefinitely:
- the round cap is reached (default 3, `review.maxRounds` in `.eagerworks/pr-review.json` overrides it) with findings still open;
- two consecutive rounds produce a substantively identical set of findings — same file, same line, same claim, same severity — the signal that a fix isn't landing and a human needs to look, not that the loop should keep retrying the same edit;
- a finding can't be confidently declined or fixed without more context from the user.

Uncapped-until-clean and a single fixed round count (no identical-findings or context-needed escape hatches) were both considered and rejected: the former risks non-termination on a fix that keeps reintroducing the bug it just closed, the latter cuts off a loop that's converging normally on round 2.

## Consequences

- A user who wants findings fixed has to ask for it explicitly every time; there's no persistent "always auto-fix" mode, which is intentional — it keeps the read-only default honest rather than becoming opt-out.
- The loop can stop with `critical`/`high` findings still open (round cap hit, or identical findings twice) — the report to the user must say so plainly rather than implying success, since "the loop ended" and "the diff is clean" are not the same claim.
- A documentation suggestion drafted mid-loop (Lens 5B) never advances or resets the round counter, so a repo that qualifies for a suggestion doesn't get an extra round it didn't ask for, and doesn't get the loop declared done prematurely because the suggestion step produced no new severity-rated finding.
- `evals/pr-review/evals.json` cases 8 and 9 (`c962806`) cover the loop's happy path (fix, re-verify, re-review, converge) and the identical-findings stop condition; case 17 covers the Lens 5B drafting step's round-cap exemption.

## Related

- [2026-08-21--documentation-decision-capture-lens](2026-08-21--documentation-decision-capture-lens.md) — introduces the Lens 5B suggestion this ADR's round-cap exemption (judgment call 4) depends on.
- [2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md) and [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) — the other precedents in this skill for stopping to ask, or disclosing, rather than silently proceeding.
- `skills/pr-review/references/workflow.md` → "Optional Fix Loop" — the operative procedure this ADR justifies.
- `evals/pr-review/evals.json` — cases 8, 9, 17.
