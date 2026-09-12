# Defer running evals in GitHub Actions

- **Date:** 2026-08-28

## Context

`evals/<name>/evals.json` (see [`docs/evals.md`](../evals.md)) defines per-skill
test cases as a `prompt` plus a checklist of `expectations`. Grading isn't
string matching — each expectation is a free-text claim about the response
("flags the missing org scope as critical", "does not invent minor style
nits"), which requires an LLM to judge semantically. Running an eval case
end-to-end therefore costs **two LLM calls**, not zero: one to produce the
response (with the skill loaded), one to grade that response against
`expectations[]`.

We considered wiring this into GitHub Actions so eval regressions surface on
every PR that touches `skills/**` or `evals/**`, the same way a normal test
suite would. Before doing that, a few things stood out:

- **Real, recurring cost.** Every run needs an `ANTHROPIC_API_KEY` secret and
  spends real API budget — run + grade, times however many eval cases exist
  across skills (17+ for `pr-review` alone). This isn't a one-time setup cost;
  it recurs on every triggering push.
- **Non-determinism as a merge gate.** LLM grading can flip a borderline case
  from pass to fail between runs. Treating that as a required, blocking check
  would produce flaky red builds unrelated to the actual change.
- **The existing tooling isn't headless.** The grading loop in
  `.agents/skills/skill-creator/` (see `docs/evals.md`) assumes an interactive
  Claude Code session — it spawns grader **subagents** as part of a live skill
  invocation, not a script you can shell out to from a CI runner. Getting this
  into Actions means writing a standalone script that calls the Anthropic API
  directly for the grading step, which doesn't exist yet.

None of these are blockers forever, but none are solved today either, and this
repo has no CI at all yet (no `.github/workflows/`).

## Decision

**Do not add a GitHub Actions workflow for evals right now.** Evals continue to
run the way they do today: on demand, via the `skill-creator` skill's
content-quality loop, when a contributor is actively iterating on a skill.

The intent is to revisit this once the above is addressed — specifically, a
headless grading script exists and there's a concrete answer for API budget.
A draft workflow is included below as a starting point for that future work;
it is **not** placed in `.github/workflows/` and should not be treated as
active.

## Consequences

- No CI gate on eval regressions for now — a skill change that breaks an
  existing eval case is only caught if someone runs the `skill-creator` loop
  manually, per `CONTRIBUTING.md`'s "Eval cases" section.
- Contributors are still expected to add eval cases for new/changed behavior
  (per `CONTRIBUTING.md` and the root `CLAUDE.md`), even though nothing runs
  them automatically yet.
- When this is revisited, the workflow should almost certainly land as a
  **non-blocking, informational** job (e.g. a PR comment with pass/fail per
  eval) rather than a required check, given the non-determinism above — and
  scoped to only trigger on `skills/**` / `evals/**` changes to control cost.

## Draft workflow (for later, not active)

Sketch of what this could look like once a headless grading script exists.
This assumes a not-yet-written `scripts/run_evals_headless.py` (or similar)
that, for a given `evals/<name>/evals.json`, runs each `prompt` with the skill
loaded, grades it against `expectations[]` via the Anthropic API, and prints a
machine-readable summary (e.g. JSON to stdout).

```yaml
name: Skill evals

on:
  pull_request:
    paths:
      - "skills/**"
      - "evals/**"

jobs:
  run-evals:
    runs-on: ubuntu-latest
    # Informational only — not a required check, since LLM grading is
    # non-deterministic on borderline cases.
    continue-on-error: true
    steps:
      - uses: actions/checkout@v4

      - name: Detect changed skills
        id: changed
        uses: tj-actions/changed-files@v44
        with:
          files: |
            skills/**
            evals/**

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r .agents/skills/skill-creator/requirements.txt

      - name: Run evals for changed skills
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          for skill in $(scripts/list-changed-skills.sh "${{ steps.changed.outputs.all_changed_files }}"); do
            python -m scripts.run_evals_headless \
              --eval-file "evals/${skill}/evals.json" \
              --skill-path "skills/${skill}" \
              --output "results-${skill}.json"
          done

      - name: Post results as PR comment
        uses: actions/github-script@v7
        with:
          script: |
            // Read results-*.json, format a pass/fail summary per skill,
            // and post/update a single PR comment. Not a required check.

      - name: Upload raw results
        uses: actions/upload-artifact@v4
        with:
          name: eval-results
          path: results-*.json
```

Open questions to resolve before this goes live: how `scripts/run_evals_headless.py`
authenticates and calls the grader (direct Anthropic API call vs. some other
mechanism), how much of `skill-creator`'s existing grading logic it can reuse
vs. needs to reimplement for headless use, and what the actual per-run cost
looks like once measured.

## Related

- [`docs/evals.md`](../evals.md) — how the eval system works today.
- [`2026-06-30--evals-separate-from-skills.md`](2026-06-30--evals-separate-from-skills.md) — why `evals/` lives outside `skills/`.
