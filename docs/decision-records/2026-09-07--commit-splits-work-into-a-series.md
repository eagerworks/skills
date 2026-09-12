# The commit skill splits a dirty working tree into a series of commits instead of one

- **Date:** 2026-09-07

## Context

Writing `skills/commit/SKILL.md` required settling what the skill actually produces when the working tree has several unrelated changes pending at once. Two failure modes sit on either side of this decision. Sweeping everything uncommitted into a single commit — the behavior of most ad hoc "commit my changes" prompting, including the pre-existing personal `commit` skill this one replaces — produces a commit message that's either a lie ("feat: add refresh endpoint" that also quietly bumps a dependency) or a vague catch-all ("various changes") that can't be reviewed, bisected, or reverted independently. The opposite failure — grouping too aggressively, or splitting at the hunk level with `git add -p` — produces commits that don't individually build, and `git add -p` in particular is an interactive prompt an agent cannot drive at all.

A second question followed: once the skill is willing to create more than one commit, does each commit in the series need its own confirmation before the next one runs? Requiring that turns "commit this" into a multi-turn back-and-forth for something the user asked for once. Requiring none risks an agent creating a long, wrong series unattended with no chance to redirect it.

A third question: what vocabulary of types and scopes does the skill write with? Defaulting to the full Conventional Commits type set regardless of the repo would let a repo whose entire history only uses four types (as this repo does — `docs`/`feat`/`fix`/`chore`, per `CONTRIBUTING.md`) suddenly start receiving `style:` or `refactor:` commits that don't match anything else in its log.

## Decision

1. **The default behavior is a series, not a single commit.** The skill groups by change intent using the ladder in `skills/commit/references/grouping.md` — an already-staged index first, then an explicit instruction, then change intent, then mechanical companions (lockfile/manifest, migration/schema), then everything left over on its own.
2. **The unit of staging is the whole file, never a hunk.** `git add -p` is never used — it's interactive and would hang an agent. A file that mixes two unrelated changes stays in one commit, and the skill says so rather than fabricating a split.
3. **The whole proposed series is shown before any commit is created, and then proceeds as one unit** — matching the precedent in `2026-09-07--create-pr-write-posture.md` for a push→create→fixup sequence the user already approved once. No per-commit confirmation gate; a partial failure mid-series is reported honestly rather than either rolled back or glossed over.
4. **The type/scope vocabulary is inferred from the repo's own `git log` before falling back to the full Conventional Commits type set**, overridable via `.eagerworks/commit.json`. A repo that has only ever used a subset of types should keep receiving commits from that same subset unless it explicitly configures otherwise.
5. **The skill never runs `git push`, `--amend`, `rebase`, `reset --hard`, or creates a branch.** It inherits the "intentional, scoped mutation exception" precedent from `2026-09-07--create-pr-write-posture.md` rather than re-litigating whether committing needs confirmation, but the exception is scoped to the commit itself — everything downstream is `create-pr`'s job.
6. **An explicit user instruction to make one commit (or to group differently) overrides the default ladder entirely.** The series is a default, not a mandate.

## Consequences

- Users asking this skill to "commit my changes" get a series of commits by default, which changes existing habits formed around the old one-shot personal `commit` skill — this is the intended improvement, not a regression, but it should be called out the first time a user notices multiple commits landing from one request.
- Because grouping depends on reading intent from a diff, a genuinely ambiguous working tree costs one clarifying question (asked once, for the whole series) rather than a wrong guess — see `skills/commit/references/grouping.md` → "When to ask instead of guess."
- The exclusion list (`.env`, credentials, `node_modules`, build output, screenshots) must always be disclosed when it triggers, following the repo's existing "a skip is never silent" rule.
- `create-pr`'s own workflow (`skills/create-pr/references/workflow.md` step 3) already told agents to defer to "the repo's own `commit` skill" — that hand-off now resolves to a real skill instead of a dangling reference.

## Related

- `2026-09-07--create-pr-write-posture.md` — the write-posture precedent this record inherits rather than re-litigates, including the naming of "a changelog-commit skill" as an anticipated future exception.
- `2026-08-21--ignore-paths-must-be-disclosed.md` — the "a skip is never silent" rule this record's exclusion list follows.
- `2026-08-21--optional-fix-loop-and-round-cap.md` — the precedent for capping a retry loop, mirrored here in the one-retry rule for a formatter hook that rewrites files.
- `skills/commit/SKILL.md` and `skills/commit/references/grouping.md` — the shipped behavior this record justifies.
