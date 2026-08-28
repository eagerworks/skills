# How evals work in this repo

This repo has two separate layers for testing skill quality: a **content layer**
(`evals/<name>/evals.json`) that defines the test cases, and a **tooling layer**
(the vendored `skill-creator` skill) that actually runs them and grades the
results. This doc explains both and how they fit together.

## Layer 1: `evals/<name>/evals.json` — the test cases

Each skill has a parallel `evals/<name>/` directory, kept **outside**
`skills/<name>/` on purpose: the skills.sh CLI ships the entire directory
containing a skill's `SKILL.md`, so anything under `skills/<name>/` reaches end
users. Eval cases are a development/test harness, not part of the shipped
knowledge base — see
[`docs/decision-records/2026-06-30--evals-separate-from-skills.md`](decision-records/2026-06-30--evals-separate-from-skills.md)
for the full rationale.

`evals/<name>/evals.json` is pure content — no code. Its shape:

```json
{
  "skill_name": "pr-review",
  "evals": [
    {
      "id": 1,
      "prompt": "The task you'd hand to an agent that has the skill loaded",
      "expected_output": "Prose description of what a correct response looks like",
      "files": [],
      "expectations": [
        "One specific, objectively-verifiable claim about the response",
        "Another specific, objectively-verifiable claim about the response"
      ]
    }
  ]
}
```

- **`prompt`** — the query/task given to the agent, exactly as a user might phrase it.
- **`expected_output`** — a narrative summary of the ideal answer, for a human skimming the file.
- **`expectations`** — a checklist of atomic, verifiable assertions. This is what a grader actually checks pass/fail against, one at a time — not the prose in `expected_output`.

Good eval cases target the decisions a skill has to get right, not trivia. For
example, `evals/pr-review/evals.json` has cases that force: correct severity
assignment, not inventing findings on a clean diff, resolving an ambiguous base
branch, respecting `.eagerworks/pr-review.json` config, stopping the fix loop on
a repeated finding, and so on — situations where a weaker prompt or a missing
skill would plausibly get it wrong.

When a change alters a skill's behavior or adds new coverage, add a
corresponding case to its `evals.json` (see `CONTRIBUTING.md`'s "Eval cases"
section and step 4 of "Adding a new skill" in the root `CLAUDE.md`).

Note: `evals/<name>/evals.json` files are just content — nothing in this repo
runs them automatically (no CI workflow triggers on them today). They're
executed on demand via the tooling described below.

## Layer 2: `skill-creator` — the tooling that runs and grades evals

`.agents/skills/skill-creator/` is a vendored skill (tracked in
`skills-lock.json`, sourced from `anthropics/skills`) and is the **only**
executable code in this repo. It's a managed dependency — don't hand-edit it as
if it were project content. It provides two distinct loops.

### a) Content-quality loop (consumes `evals/<name>/evals.json`)

1. **Run** — each eval's `prompt` is run twice: once with the skill available,
   once as a baseline without it (in parallel, via subagents).
2. **Grade** — a grader subagent (`agents/grader.md`) checks each response
   against every item in `expectations[]` and writes `grading.json` per run,
   with `text` / `passed` / `evidence` fields per expectation.
3. **Aggregate** — `scripts/aggregate_benchmark.py` rolls the per-run grades up
   into `benchmark.json`: pass rates, timing, token usage, broken down per eval
   and per configuration (with-skill vs. baseline).
4. **Review** — `eval-viewer/generate_review.py` serves a local HTML viewer that
   shows the with-skill and baseline outputs side by side, plus the
   quantitative benchmark, so a human can judge quality and leave feedback
   (saved to `feedback.json`).

This loop is what you reach for after editing a skill: it tells you whether the
skill measurably improves the agent's response versus not having it, on the
cases in `evals.json`.

### b) Trigger-accuracy loop (independent of `evals.json`)

A separate concern: does the skill's frontmatter `description` cause Claude to
actually load the skill when it should (and *not* load it when it shouldn't)?

1. Generate ~20 trigger queries — a mix of should-trigger and should-not-trigger
   — and review them with the user via the `assets/eval_review.html` template.
2. `scripts/run_loop.py` runs the queries against real `claude -p` sessions
   (splitting 60% train / 40% held-out test), measures the trigger rate, and
   iterates on the description (up to 5 rounds) to improve it — picking the
   best description by test-set score to avoid overfitting.

This loop tunes `SKILL.md`'s frontmatter `description`, not the skill's body
content, and uses its own throwaway query set rather than `evals/<name>/evals.json`.

## Summary

| | Lives in | Format | Purpose |
|---|---|---|---|
| `evals/<name>/evals.json` | repo root, per skill | JSON: prompt + expected_output + expectations | The test cases — what "correct" looks like |
| `skill-creator` content loop | `.agents/skills/skill-creator/` | Python + subagents | Runs `evals.json` cases with/without the skill, grades, benchmarks, and renders a comparison viewer |
| `skill-creator` trigger loop | `.agents/skills/skill-creator/` | Python + `claude -p` | Optimizes `SKILL.md`'s `description` for correct triggering, using its own query set |

In short: `evals/<name>/evals.json` is the *what to test*; `skill-creator` is the
*how to run it*.
