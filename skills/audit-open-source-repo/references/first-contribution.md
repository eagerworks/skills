# Audit Open Source Repo — First Contribution Dry Run

Section 4 of the report is a **view over grades already assigned** — strictly weaker than a new finding. It introduces no check, no grade, no Plan row, and never moves the verdict. Its value is rendering the rubric in the order a human actually experiences it, and naming the exact point where the path stops. Output shape: `references/output-format.md` → section 4.

## The eight stages

The stage → check map is **fixed** — never re-decided per run.

| # | Stage | Inspects | Maps to |
|---|---|---|---|
| 1 | **Land** | README's first 30 lines, LICENSE, the CONTRIBUTING link, `gh repo view --json description,topics` | 1.1–1.4, 1.8, 2.1 |
| 2 | **Decide it's alive** | last default-branch commit date, latest release date, the stated status | 1.5, 10.2, 10.8 |
| 3 | **Set up** | the documented setup path resolved **on paper**: every command, script, file, service and env var it names, in order, stopping at the first token that doesn't exist | 2.8, 4.1–4.6, 4.8 |
| 4 | **Pick something** | open counts for the entry-point labels; up to 3 such issues read in full and reported as actionable/claimed/stale, by number | 5.5, 5.6, 5.7 |
| 5 | **Find where the code goes** | the most recently merged **feature** PR (or a representative feature commit): the file set it touched, and whether the repo's own docs would have led a stranger to those files | 6.1–6.4 |
| 6 | **Meet the standards** | the documented check-only commands run once on the untouched checkout — exit codes and wall times; whether one aggregate command covers them; whether a hook would have caught them pre-push | 7.1–7.5, 7.7 |
| 7 | **Prove it works** | the documented test command run once, non-interactively, with a timeout — exit code and wall time; whether a focused run is documented | 8.1–8.5 |
| 8 | **Open the PR, get the gate green** | the PR template, then the workflows: does the gate trigger on a fork PR, does it need secrets, is the default branch green, what is the median run time | 5.4, 9.1–9.3, 9.6–9.8 |

**Stage 5 is the honest answer to "can they find where a feature goes."** The audit does not imagine a feature and role-play building it; it takes a feature that *actually shipped*, lists the files it touched, and asks whether the documentation names them. That is a measurement, not a guess.

## Result vocabulary

Plain words, never emoji — nothing may look like a fifth grade.

| Word | Means | Derivation (mechanical) |
|---|---|---|
| **Clears** | The stage works as documented | every check mapped to the stage is 🟢 |
| **Costs them** | They get through, slower or wronger | the stage's worst mapped check is 🟡 |
| **Stops here** | The documented path cannot be followed | the stage contains a 🔴 |
| **Can't tell** | Needs a setting, a dashboard, or a person | the stage's only non-🟢 checks are ⚪ |

## What it may execute

Exactly the same allow-list as `references/audit-workflow.md` → Phase 4, no wider: check-only format/lint/typecheck and the test command, each once, non-interactive, with a timeout, and only when dependencies are already installed and `runCommands` isn't `false`. It **never** runs setup, bootstrap, install, migrate, seed, `docker compose`, deploy, or publish; never forks, clones, branches, pushes, comments, labels, or opens anything. Stage 3 is graded by **resolution**, not execution — that is the whole point of it. Missing dependencies make stages 6–7 **Can't tell** with "dependencies not installed; the audit never installs" as the evidence, never 🟡.

## Honesty rules

- Every cell is a fact: a path, a line, a command with its exit code and duration, a count, a date. No cell may begin "a contributor would probably…".
- The first-break sentence is required even when nothing breaks: "No stage stops; the earliest cost is stage 4 (…)."
- It never fabricates an issue number, a file path, a maintainer name, or an error message it didn't see.
- It never simulates *making* the change, never opens anything, and never scores tone ("the README is welcoming"). The graded checks already carry every judgment the report is entitled to.
- It adds no plan rows and changes no grade. Every stage cites the check ids it renders; a stage that wants to say something no check covers means the rubric has a hole — fix `references/dimensions.md`, not the dry run.
- `dryRun.enabled: false` omits the section entirely and the footer says `dry run: disabled by config` — never a silent skip.
