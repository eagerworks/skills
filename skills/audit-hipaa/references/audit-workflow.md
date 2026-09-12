# HIPAA — Audit Workflow

> The six phases end to end, the exact read-only command list, how to triage a large codebase,
> and how to write the report file. This is the skill's procedural spine — `SKILL.md` only
> summarizes it.

## Table of Contents

1. [Phase 1 — Scope](#phase-1--scope)
2. [Phase 2 — Locate PHI](#phase-2--locate-phi)
3. [Phase 3 — Technical safeguards](#phase-3--technical-safeguards)
4. [Phase 4 — Audit controls](#phase-4--audit-controls)
5. [Phase 5 — Infrastructure](#phase-5--infrastructure)
6. [Phase 6 — Write the report](#phase-6--write-the-report)
7. [Triaging a large codebase](#triaging-a-large-codebase)
8. [Allowed commands](#allowed-commands)

---

## Phase 1 — Scope

Follow `SKILL.md` → Scoping. Classify the org's role (covered entity / business associate /
subcontractor / neither) and confirm before proceeding — see `references/phi-identification.md`
→ "Does HIPAA apply at all?" for the boundary cases. If the classification is genuinely
ambiguous from the code and conversation alone, ask the user directly rather than picking a side.

## Phase 2 — Locate PHI

Run the starting greps from `SKILL.md`, then read every match's surrounding context — a
PHI-shaped column name doesn't confirm PHI actually flows there, and a generic-looking model can
still carry PHI in a free-text field. Build a working list of every PHI entry/rest/exit point;
this list drives every phase after it. `assets/phi-inventory.md` is the fillable version of this
list — use it as the working document, not just the final deliverable.

## Phase 3 — Technical safeguards

Walk §164.312 (`references/technical-safeguards.md`) against the PHI inventory from Phase 2:
access control, integrity, person/entity authentication, transmission security. For each
Addressable spec with no implementation found, look for a documented equivalent alternative
before grading it a gap — its absence, not just the spec's absence, is what makes it a finding
(`SKILL.md` Gotcha 1).

## Phase 4 — Audit controls

`references/audit-controls.md` in full. For each PHI-touching route/controller found in Phase 2,
confirm both read and write access are logged, check retention, and check the audit log doesn't
itself duplicate PHI content into a less-controlled location.

## Phase 5 — Infrastructure

`references/infrastructure.md`. Confirm encryption at rest and in transit, network isolation, and
that every vendor PHI actually reaches (Phase 2's inventory) is either BAA-eligible-and-covered or
flagged as a blocker if it isn't confirmed. This phase leans on config files (`terraform/`,
`cloudformation/`, `docker-compose.yml`, CI/CD secrets config) as much as application code.

## Phase 6 — Write the report

1. Resolve `docs/hipaa-audits/` at the repo root. **Create the directory if it doesn't exist** —
   this is the one directory-creation action this otherwise read-only skill takes by design.
2. Filename: `YYYY-MM-DD-<slug>.md`, using today's date and a short kebab-case slug describing
   the audit's scope (e.g. `2026-08-28-full-audit.md`, `2026-08-28-llm-integration-review.md`).
3. **Never overwrite an existing dated report.** Each audit is its own dated record — if a report
   for today's date and slug already exists, append a numeric suffix (`-2`, `-3`) rather than
   replacing it. See `docs/decision-records/` for why: the report is itself compliance evidence
   under the 6-year retention rule (`references/audit-controls.md`), and overwriting destroys the
   history that rule expects.
4. Fill `assets/audit-report.md`'s shape: Blockers, then Risks, then Needs-a-human, then a Pass
   summary, then Assumptions & unverified — every finding cites `file:line` and a §-numbered
   safeguard.
5. This is the **only** file the audit writes. Everything else stays read-only unless the user
   explicitly asks for fixes to be applied.

## Triaging a large codebase

When the codebase is too large to review every file with equal depth, prioritize in this order:
PHI data models and their migrations, controllers/routes that read or write those models,
logging/error-tracking configuration, outbound API/LLM client code, then everything else. State
explicitly in the report which areas got a lighter pass — a partial audit should never read as if
it were exhaustive, same principle `pr-review` applies to large diffs.

## Allowed commands

Read-only inspection only. The greps and cloud CLI `describe`/`get`/`list` calls throughout the
other reference files are the model — never run a command that creates, modifies, or deletes a
cloud resource, and never run a mutating `terraform apply`/`aws ... put-*`/`aws ... create-*` — the
audit describes what exists, it doesn't change it. The one write action in the entire skill is
creating the report file and, if absent, the `docs/hipaa-audits/` directory itself.
