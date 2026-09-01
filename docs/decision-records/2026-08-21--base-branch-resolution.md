# Resolve the `pr-review` base branch from evidence, never a silent default

- **Date:** 2026-08-21

## Context

Before this decision, `pr-review`'s `SKILL.md` resolved the base branch with a single line: fall back to `gh repo view --json defaultBranchRef`, or whichever of `main`/`master`/`develop` exists on `origin`, unless `.eagerworks/pr-review.json` set `baseBranch`. That is a guess. It never looked at the open PR the branch under review already has, and never looked at where the branch or worktree was actually forked from. A wrong base silently changes what the diff even is — every one of the four lenses then runs against the wrong range, and the report reads as authoritative while being wrong about its own scope.

We traced what real signals are available, from inside this repo's own worktree setup, before picking an order:

| Command | Result here | Lesson |
|---|---|---|
| `gh pr view --json baseRefName` | `main` (PR #4) | Strongest signal, and cheap. |
| `git reflog show <branch> \| grep 'Created from'` | `Created from d6af7b6…` (a SHA) | Fork point exists but often isn't a branch *name*. |
| same, on `feat/rest-api-design-skill` | `Created from HEAD` | Sometimes useless entirely. |
| `git branch -a --contains d6af7b6` | 7 branches | Ambiguity is the normal case in a multi-worktree repo, not the exception. |
| `git rev-parse --abbrev-ref @{u}` | `origin/worktree-code-review-skill` | The branch's *own* remote ref — a trap, never the base. |
| `git symbolic-ref refs/remotes/origin/HEAD` | `fatal: not a symbolic ref` | The "obvious" default-branch lookup fails on an ordinary clone. |

Given that, two questions needed a explicit answer rather than an implicit one:

1. When the branch under review already has an open PR whose base disagrees with `.eagerworks/pr-review.json`'s `baseBranch`, which wins?
2. When neither an open PR nor an unambiguous fork point exists, does the skill still fall back to a repo default, or stop and ask?

## Decision

Resolve the base branch with a five-rung ladder, stopping at the first rung that yields an unambiguous answer:

0. The user named it explicitly (a branch, `review PR #N`, `--staged`, working tree) — nothing below runs.
1. The open PR for the branch under review (`gh pr view --json baseRefName`). **This outranks the config file** — the PR's base is where the code actually merges; a checked-in `baseBranch` is a stale-able declaration, not a fact about this specific review.
2. `.eagerworks/pr-review.json` → `baseBranch`, if no open PR exists.
3. The fork point of the branch/worktree, via `git reflog show <branch>`. If it resolves to a bare SHA (or `HEAD`, or the reflog is gone), widen with `git branch -a --contains <sha>` and use the result only if exactly one candidate survives.
4. Ask the user, listing the candidates actually found — fork-point survivors, sibling-worktree branches, and the repo default as one labelled option among others, never pre-selected or auto-applied.

The full procedure, including the anti-patterns above (`@{u}`, an unset `origin/HEAD`, many-candidate `--contains` results), lives in `skills/pr-review/references/base-branch.md`. The silent `main`/`master`/`develop` fallback is removed entirely: with no reliable signal, the skill always asks instead of guessing.

## Consequences

- A review can now pause to ask when the base is genuinely ambiguous, costing a round-trip in that case — but it never silently reviews the wrong range.
- Repos that pinned `baseBranch` in `.eagerworks/pr-review.json` may see it overridden whenever the branch under review has its own open PR against a different base; this is intentional, not a bug.
- Every rung stays read-only (`gh pr view`, `git reflog`, `git branch --contains`, `git worktree list`), consistent with the skill's existing read-only-by-default rule.
- The `code-reviewer` subagent (`skills/pr-review/assets/code-reviewer.agent.md`) has no way to prompt the user — when it reaches rung 4 it must stop and report the candidates in its findings rather than pick one.

## Related

- [2026-06-30--evals-separate-from-skills](2026-06-30--evals-separate-from-skills.md) — why `evals/pr-review/evals.json` sits outside the shipped skill; the cases covering this decision land there, not in `skills/pr-review/`.
- `skills/pr-review/references/base-branch.md` — the operative procedure this ADR justifies.
