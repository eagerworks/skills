# `create-pr` mutates by design, deliberately breaking from this repo's read-only-by-default posture

- **Date:** 2026-09-07

## Context

Every skill in this repo so far treats mutation as the exception. `pr-review` is read-only with exactly two carve-outs — an optional fix loop that only runs on an explicit ask, and posting the finished report to the PR (`skills/pr-review/SKILL.md` → Critical Gotchas #1: "Never edit, create, or delete a file, and never `git commit`, `git push`, or open a PR on your own initiative"). `loop-engineering-audit` has exactly one write, a fixed report path, and never runs setup/migrate/deploy commands. Writing `skills/create-pr/SKILL.md` forced a concrete decision: does this new skill inherit that same "read-only unless told otherwise" default, or is it a different kind of skill?

It can't inherit it literally — a skill whose entire purpose is to commit, push, and open a pull request has no meaningful read-only mode to default to. But without some line, "mutates by design" risks sliding into "mutates without asking," which is exactly the failure mode the repo's existing posture exists to prevent. The concrete judgment call was where the line falls: what may happen without a fresh confirmation each time (inspecting `git status`/`git diff`, obviously never needs one), what always needs the user's go-ahead (`git push`, `gh pr create`/`gh pr edit`), and what's genuinely ambiguous — a request like "help me write a PR description" names the description, not the act of opening it, and treating that as implicit permission to push and create would surprise a user who only wanted a draft to review.

A second question followed directly: once opening is clearly wanted, does each individual mutating step (push, then create, then any assignee/label fixup) need its own separate confirmation? Requiring that would make the skill feel like it's constantly asking permission for a single coherent action the user already approved once; requiring none would make the "confirm when ambiguous" rule toothless, since a chain of unconfirmed steps is functionally the same as no confirmation.

## Decision

1. **`create-pr` is a deliberate exception to this repo's read-only-by-default posture**, not a loosening of it. The exception is scoped narrowly: it applies to a skill whose entire job is to produce the artifact in question (a PR), not to skills that merely touch git/gh as a side effect of reviewing or auditing.
2. **Read-only inspection never needs confirmation**: `git status`, `git diff`, `git log`, `gh pr view`, `gh --version`, `gh label list` run freely as part of ordinary preflight and drafting.
3. **`git push` and `gh pr create`/`gh pr edit` require the user to have clearly asked for the PR to be opened or updated**, not merely for help writing its content. When that's ambiguous, the skill asks before running either — see `skills/create-pr/SKILL.md` → "Mutation posture" and Critical Gotcha #1.
4. **Once opening is clearly requested, the push → create → fixup sequence proceeds as one unit without a second confirmation per step** — see `skills/create-pr/references/workflow.md` → "Standard flow." Re-confirming before each individual command in a sequence the user already approved would be noise, not safety.
5. **A branch that already has an open PR is always edited (`gh pr edit`), never duplicated with a second `gh pr create`** — this is not a confirmation question, it's a correctness one, and it holds regardless of how step 3 resolves.

## Consequences

- A future skill that primarily *reads* (reviews, audits, analyzes) inherits `pr-review`'s read-only-by-default posture, not `create-pr`'s — this record does not loosen the repo-wide default, it names one narrow exception to it.
- A future skill whose entire purpose is also to produce a mutating artifact (e.g., a release-notes-and-tag skill, a changelog-commit skill) can point at this record as the precedent for being an intentional, scoped exception, rather than re-litigating "should this skill be read-only" from scratch.
- `create-pr` must never treat "the user asked me to write a description" as license to also push and open the PR — that boundary is load-bearing and any future edit to `SKILL.md`'s mutation posture section should preserve it.
- The workflow document is the operative source for exactly which commands run without a fresh confirmation and which don't; if that sequence changes, this record's Decision section should be revisited rather than left to silently drift from the shipped behavior.

## Related

- `skills/pr-review/SKILL.md` → Critical Gotchas #1 — the read-only-by-default precedent this record deliberately departs from, and the boundary of its own two carve-outs.
- `skills/create-pr/SKILL.md` → "Mutation posture" — the operative statement of what this record decides.
- `skills/create-pr/references/workflow.md` — the standard flow that treats push → create → fixup as one confirmed unit.
