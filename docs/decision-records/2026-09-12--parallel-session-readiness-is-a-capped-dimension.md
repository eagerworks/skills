# Parallel-session readiness is a graded dimension whose checks never block the verdict

- **Date:** 2026-09-12

## Context

Loop engineering at scale means several agent sessions working the same repo at once, each in its own `git worktree`. The failure modes there are specific and, unlike most of what a repo review has to guess at, **verifiable from the repo itself**: a fresh worktree with no `.env` because nothing regenerates it; a hardcoded `app_test` database that two suites truncate out from under each other; a fixed port or `container_name:` that the second session collides with; a hook manager installed only into the primary clone's `.git/hooks`, silently inert in every linked worktree.

Three ways to place this in the skill, each with a real failure mode on the other side:

1. **Advisory, alongside the automation-map/loop-recommendation output** (see the sibling record). Rejected: unlike "which loops should we run," worktree readiness *is* gradeable against concrete rules with a 🟢/🟡 evidence bar — treating it as advisory would throw away exactly the kind of check this rubric is good at, for no reason but convenience.
2. **Folded into dimension 2 (Reproducible environment).** Dimension 2 already asks "clone to passing tests without a human"; parallel readiness asks "N clones to N passing test runs without collision" — a different question that fails differently, and dimension 2 rolls up to its worst check, so the parallel-session checks would be invisible behind whatever dimension-2 problem is worse.
3. **Its own dimension.** The natural fit, but with a real risk: a 🔴 in it would declare a repo "Not ready" when a single agent loop runs on it perfectly well today. Parallelism is a throughput property, not a can-it-run-at-all property, and the verdict's whole value is that "Not ready" means the loop literally cannot run unattended.

## Decision

1. **Dimension 8 — Parallel-session readiness**, seven checks (8.1–8.7) in `references/rubric.md`, in the same `Check / 🟢 when / 🔴 or 🟡 when` format as the other seven: gitignored-but-required files reproducible, no clone-location assumptions, stateful resources (DB/Redis/index) isolate per worktree, ports/PIDs/containers don't collide, per-worktree setup cost is bounded, git tooling survives a linked worktree, the parallel workflow is documented.
2. **No check in dimension 8 is ever 🔴** — the same posture already used for check 4.5 ("informational, never 🔴"). It can move the verdict from **Ready** to **Partially ready**, never to **Not ready**. This resolves judgment call 3: the dimension gets real grading power without corrupting what "Not ready" means.
3. **The audit never runs `git worktree add`.** Creating a worktree writes files and mutates `.git` — a second write, which this skill's write posture (`2026-08-28--audit-report-saved-to-docs.md`) forbids. Dimension 8 is graded entirely from source: setup scripts, `config/database.yml`-style files, `docker-compose*.yml`, `.gitignore`. `git worktree list` is read-only and allowed — it tells the audit whether the team already works this way, without creating anything. Anything only a live worktree could settle is ⚪.
4. **8.3 (a shared test database) is the single most damaging check on the list**, called out explicitly: it makes parallel test runs *silently wrong*, not merely slow or inconvenient, and it's the first thing to rule out before concluding a suite is flaky (rubric check 3.6) — a repo review would otherwise misdiagnose a collision as a flaky test and never see the real cause.
5. **Gap-leverage ordering** in the Work Plan (`references/audit-workflow.md` Phase 4) becomes `1, 3, 2, 6, 7, 8, 4, 5` — parallelism pays off once a team runs more than one session, so it sits after guardrails and ahead of coverage/task-definition, which mostly pay off within a single loop.
6. **`dimensions.parallelSessions.enabled`** joins the existing `dimensions.*` config block for a repo whose policy is a single clone, never worktrees — disclosed in the footer exactly like any other disabled dimension.
7. Check 8.7 (a documented parallel workflow) needed something to grade against, so `references/agent-docs.md`'s required-sections checklist and the `assets/AGENTS.example.md` starter both gained a matching **Worktrees / parallel sessions** section.

Rejected: making 8.3 a 🔴 despite being tempting — a shared test database is a genuinely bad bug, but it stops zero single-session loops, and a 🔴 here would mean "Not ready" no longer reliably means "the loop can't run." Also rejected: a separate `worktrees` config block — the existing `dimensions.*.enabled` toggle already covers the one thing a repo needs to say (skip this dimension entirely).

## Consequences

- A repo graded 🟢 Ready before this change may now grade 🟡 Partially ready if it has a shared test database, a fixed port, or an undocumented worktree convention. That's not a regression in the audit — it's the honest answer to a question the audit wasn't asked before ("can agents work on this repo autonomously" *in parallel*), and this record exists so the change in grading isn't mistaken for a bug.
- The verdict keeps exactly the meaning it had: **Not ready = the loop cannot run unattended, full stop.** Parallel-session gaps are a ceiling on throughput, not a floor under correctness.
- The dimension count moves from seven to eight everywhere it was hardcoded — `SKILL.md`, the skill's own `README.md`, the root `README.md`, `CLAUDE.md`, `references/audit-workflow.md`'s leverage order, and `evals/loop-engineering-audit/evals.json` case 6's "all seven dimensions" claim.
- A shared test database now has a named check (8.3) to cite before recommending the flaky-test-quarantine loop from the sibling advisory catalog — quarantining tests that are actually colliding on shared state would have treated the symptom.
- Read-only worktree probes (`git worktree list`, `git config core.hooksPath`, grepping for hardcoded paths and ports) were added to Phase 1 of `references/audit-workflow.md`, not Phase 2 — they cost nothing to run and need no allow-list, unlike the lint/test/build commands Phase 2 gates.

## Related

- [`2026-08-28--audit-report-saved-to-docs.md`](2026-08-28--audit-report-saved-to-docs.md) — the one-write rule that makes `git worktree add` out of bounds for this audit.
- [`2026-09-12--loop-recommendations-are-advisory.md`](2026-09-12--loop-recommendations-are-advisory.md) — the sibling record for the other half of this change; its catalog's concurrency floor depends on dimension 8.
- `skills/loop-engineering-audit/references/rubric.md` — dimension 8 in full.
- `skills/loop-engineering-audit/references/audit-workflow.md` — the Phase 1 worktree probes and the updated leverage order.
- `skills/loop-engineering-audit/references/agent-docs.md`, `skills/loop-engineering-audit/assets/AGENTS.example.md` — the worktree section check 8.7 grades against.
