# PR Review — Workflow

## Standard Review (Read-Only)

1. **Resolve the scope** — see `SKILL.md` → Scope Detection. Get the full diff since the branch
   forked from its base, not just the last commit.
2. **Read the repo's conventions and config** — `AGENTS.md`/`CLAUDE.md`, and
   `.eagerworks/pr-review.json` if present (`references/config.md`).
3. **Read every touched file in full**, not just the changed hunks — the diff alone hides
   context (existing scoping patterns, sibling tests, error-handling style) needed to judge
   lenses 1–3 correctly.
4. **Pass all four lenses** from `references/rubric.md` over the change.
5. **Write the report** — see `references/output-format.md` for the format.

## Reviewing Against an Issue or PR

When there's an issue or PR with a description, pull its acceptance criteria and use them as
the primary input for lens 4 (test coverage):

```bash
gh issue view <N> --json title,body
gh pr view <N> --json title,body
```

If there is no issue and no PR description, fall back to reviewing every new behavior the diff
introduces against its own tests — the same lens, just without a written acceptance-criteria
list to check off.

## Optional Fix Loop

Only run this when the user explicitly asks for findings to be fixed, not by default. Read-only
review stays the default behavior described in `SKILL.md`.

1. **Review** — produce the findings as usual.
2. **Fix** — implement a change for every `critical`/`high` and `minor` finding, or decline one
   only by citing a specific documented repo convention the code actually follows, or by
   demonstrating the claimed failure path can't occur. Record the decline reason in one
   sentence.
3. **Re-verify** — re-run this repo's own local checks (tests, linter, type checker — see
   `references/config.md` → `localChecks`) against the new changes.
4. **Re-review** — review only what changed in this round against the same four lenses.
5. **Repeat or stop:**
   - Any `critical`/`high` finding in this round → go back to step 1.
   - This round was minor-only, or returned zero findings → the loop ends.

**Round cap** — default 3 rounds (`review.maxRounds` in `.eagerworks/pr-review.json` overrides
it). Stop and report to the user, rather than continuing indefinitely, when:
- the round cap is reached with findings still open;
- two consecutive rounds produce a substantively identical set of findings — the fixes aren't
  landing and a human needs to look;
- a finding can't be confidently declined or fixed without more context from the user.

## Posting to a PR

Only if the user explicitly asks to post the review. Confirm before posting — a PR comment is
externally visible and, once posted, may be seen even if later edited or deleted.

```bash
gh pr comment <N> --body-file <report.md>
```

One line per finding in the posted comment, however long — never hard-wrap it; let GitHub wrap
it. See `references/output-format.md` for the exact line format.

## Reviewing a Large Diff

When the diff is too large to review every line with equal depth, prioritize in this order:
schema migrations, auth/scoping guards, state-transition logic, then everything else. Say
explicitly in the report which areas got a lighter pass so the reader knows what wasn't covered
in depth — never let a partial review read as if it were exhaustive.
