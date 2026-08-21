---
name: pr-review
description: >
  Expert code reviewer for Ruby on Rails and Node/TypeScript codebases. Use this skill whenever the user: asks to review a branch, PR, diff, or working-tree changes before merging; wants a second opinion on their own code before opening a PR; asks "what's wrong with this diff" or "is this ready to merge"; wants findings posted as PR comments; asks to check for correctness bugs, security or multi-tenant scoping gaps, missing test coverage against acceptance criteria, or violations of the repo's own conventions; or wants a fix applied for review findings. Also use when the user says things like "review my branch", "look over this PR", "check this diff before I push", or "did I miss anything here" — even if they don't name the skill.
---

# PR Review Skill

Reviews a diff — a branch, a PR, staged changes, or working-tree changes — against four fixed lenses: correctness, security & data integrity, repo-convention conformance, and test coverage. Returns structured, evidence-backed findings with severities. This is **not** a linter and does **not** replace CI: skip anything a linter or type checker already catches, and don't re-derive what a failing test already proves.

By default this is a **read-only** review: no file is edited, no commit is made, nothing is pushed. See `references/workflow.md` if the user explicitly wants findings fixed automatically.

## Scope Detection — Do This First

Before reading a single line of the diff, resolve exactly what is being reviewed. In order of how requests are usually phrased:

```bash
# "review my branch" / "review this PR" — the normal case: full branch vs. its base
git diff origin/<base-branch>...HEAD      # note the THREE dots — diff since the branch forked

# "review what I'm about to commit"
git diff --staged

# "review my uncommitted changes"
git diff

# "review PR #<N>" on GitHub
gh pr diff <N>
gh pr view <N> --json body,title       # pull the description / acceptance criteria too
```

Infer the base branch if the user doesn't name one: `gh repo view --json defaultBranchRef` or fall back to whichever of `main`/`master`/`develop` exists on `origin`. If a `.eagerworks/pr-review.json` config is present (see `references/config.md`), its `baseBranch` wins.

**The most common mistake is reviewing `HEAD~1..HEAD` instead of the full branch** — that only shows the last commit and silently skips everything the branch actually changed. Always use the three-dot range against the base, not the last commit.

## Read Before Reviewing

Read these before forming any opinion — lens 3 (repo-convention conformance) is a guess without them:

1. `AGENTS.md` / `CLAUDE.md` at the repo root — the project's own stated conventions.
2. `.eagerworks/pr-review.json`, if present — see `references/config.md` for its schema and how it layers with the two files above.
3. The full contents of every file the diff touches, not just the changed hunks — the diff alone hides the surrounding context (existing scoping patterns, error handling style, sibling tests) needed to judge lenses 1–3 correctly. On a very large diff, see `references/workflow.md` → "Reviewing a Large Diff" for how to triage instead of reading everything at equal depth.

## The Four Lenses

| Lens | Checks |
|---|---|
| 1. Correctness | Logic errors, edge cases, off-by-ones, unhandled states, races, wrong error handling |
| 2. Security & data integrity | Auth/tenant scoping, injection, secrets, unsafe migrations |
| 3. Repo-convention conformance | Violates `AGENTS.md`/`CLAUDE.md` or an evident surrounding pattern |
| 4. Test coverage | Every acceptance criterion (or new behavior) has a test that actually asserts it |

Full detail, decision rules, and Rails + Node/TS examples for each lens: `references/rubric.md` — read it before writing the report, it is the authoritative checklist.

## Severity at a Glance

Three labels — `critical`, `high`, `minor` — plus "not a finding" for anything without a concrete failure scenario or violated convention behind it. The full ladder and what qualifies each finding for its label lives in `references/rubric.md` — don't re-derive it here.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The four lenses in full, with Rails + Node/TS examples, severity ladder, conservatism rule | `references/rubric.md` |
| Running a review end-to-end, reviewing against an issue, the optional fix loop, posting to a PR | `references/workflow.md` |
| The markdown report format and the machine-parseable `### FINDINGS` block | `references/output-format.md` |
| The optional `.eagerworks/pr-review.json` config schema | `references/config.md` |

Copyable templates live in `assets/`:
- `assets/pr-review.example.json` — starter config, with a Rails and a Node example
- `assets/code-reviewer.agent.md` — optional Claude Code subagent wrapper around this skill

## Critical Gotchas

1. **Read-only unless asked otherwise.** Never edit, create, or delete a file, and never `git commit`, `git push`, or open a PR on your own initiative. `git`/`gh` are for inspection only by default. The two exceptions are explicit user requests documented in `references/workflow.md`: the optional fix loop, and posting the report with `gh pr comment`.

2. **A finding needs evidence — see `references/rubric.md` → Conservatism.** No concrete failure scenario or cited convention means it's not a finding, no matter how confident the preference.

3. **Code that follows a documented convention is never a finding**, even if a different style would generally be preferable.

4. **"Already handled" is not evidence.** If a comment, commit message, or the diff's author claims something is intentional or already covered, verify it against the actual files before accepting the claim.

5. **Always diff the full branch against its base**, not just the last commit — see Scope Detection above.

6. **If you can't tell whether something is a real bug without running code, say so in the finding** rather than guessing either way.

7. **Never pad the list to look thorough.** Zero findings is a valid, correct result.
