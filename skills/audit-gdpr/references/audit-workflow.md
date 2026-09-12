# GDPR — Audit Workflow

> The six phases end to end, the exact read-only command list, how to triage a large codebase,
> and how to write the report file. This is the skill's procedural spine — `SKILL.md` only
> summarizes it.

## Table of Contents

1. [Phase 1 — Scope](#phase-1--scope)
2. [Phase 2 — Locate personal data](#phase-2--locate-personal-data)
3. [Phase 3 — Lawful basis & transparency](#phase-3--lawful-basis--transparency)
4. [Phase 4 — Data subject rights](#phase-4--data-subject-rights)
5. [Phase 5 — Security, transfers & processors](#phase-5--security-transfers--processors)
6. [Phase 6 — Write the report](#phase-6--write-the-report)
7. [Triaging a large codebase](#triaging-a-large-codebase)
8. [Allowed commands](#allowed-commands)

---

## Phase 1 — Scope

Work through `SKILL.md`'s three scoping gates in order, and don't skip past a "no" answer:

1. **Territorial scope (Art 3).** If the org has no EU/EEA establishment, doesn't offer goods or
   services to people in the Union, and doesn't monitor their behaviour, say so plainly and stop
   — do not produce a graded report. This is as important a finding as any 🔴: telling a team
   GDPR doesn't apply to them, correctly, saves them from treating an irrelevant regulation as a
   blocker on unrelated work.
2. **Controller / processor / joint controller (Art 4(7)/(8), Art 26).** Read the product's own
   relationship to its data — whose purposes determine the processing. A B2B tool processing its
   customers' end users' data is a processor to that customer; audit against Art 28 obligations,
   not the controller article set. If genuinely ambiguous, ask rather than guess — the two
   article sets don't overlap enough to average between them.
3. **Where personal data flows.** Run the grep recipes in `SKILL.md` across the whole repo, not
   just the models directory — the highest-yield findings are usually in logging, error tracking,
   and outbound API calls (`references/personal-data-in-code.md`), not the schema.

## Phase 2 — Locate personal data

Beyond the initial greps, walk:
- Data models and migrations — every column, not just ones with obviously PII-shaped names
  (a `notes` or `metadata` free-text/JSON column is a common special-category leak point)
- Application logs and the logging framework's redaction config
- Error tracker initialization and its `beforeSend`/scrubbing configuration
- Analytics/telemetry SDK `.identify()`/`.track()` call sites
- Outbound HTTP clients calling third-party or LLM APIs
- Consent/cookie management library config and the scripts it gates
- Export, backup, and data-warehouse sync jobs
- Seed files, factories, and fixtures

Build a running inventory as you go — this becomes the input to `assets/ropa-template.md` and
`assets/gap-matrix.csv` in Phase 6, and to the special-category check in
`references/personal-data-identification.md`.

## Phase 3 — Lawful basis & transparency

For each processing activity found in Phase 2, resolve: what Art 6(1) basis applies, whether an
Art 9(2) condition is needed and present, and whether the privacy notice (Art 13/14) actually
discloses this activity, its basis, and its retention. Check consent flows specifically against
the Art 7 conditions in `references/lawful-basis-and-consent.md` — most failures here are
structural (bundled checkboxes, no symmetric withdrawal path) and visible directly in the
frontend code, not just the copy.

## Phase 4 — Data subject rights

For each right in `references/data-subject-rights.md`, find the code path (if any) that
implements it and trace it to its actual endpoint, not just its existence:
- Does an erasure request reach every place personal data was found in Phase 2, or only the
  primary datastore?
- Does an access/export endpoint include the Art 15(1) metadata, or just the raw data rows?
- Is there any request-handling path with the Art 12(3) one-month clock enforced at all, or is
  DSAR handling an unmanaged support-ticket queue?

This phase is usually where the report's most concrete 🔴 findings come from — the gap between "a
delete button exists" and "erasure actually reaches every copy" is large and consistent across
codebases.

## Phase 5 — Security, transfers & processors

Walk `references/security-and-breach.md`'s Art 32 checklist against what's actually configured
(encryption, access control, backup testing, a documented review cadence), then
`references/transfers-and-processors.md` for every third-party vendor and every cross-border data
flow found in Phase 2 — does each vendor receiving personal data have an Art 28(3) contract, does
each cross-border transfer have an adequacy decision or an Art 46 safeguard, and is remote access
from outside the EU/EEA accounted for as a transfer, not just physical data location.

## Phase 6 — Write the report

1. Resolve `docs/gdpr-audits/` at the repo root. **Create the directory if it doesn't exist** —
   this is the one directory-creation action this otherwise read-only skill takes by design.
2. Filename: `YYYY-MM-DD-<slug>.md`, using today's date and a short kebab-case slug describing
   the audit's scope (e.g. `2026-09-12-full-audit.md`, `2026-09-12-llm-integration-review.md`).
3. **Never overwrite an existing dated report.** Each audit is its own dated record — if a report
   for today's date and slug already exists, append a numeric suffix (`-2`, `-3`) rather than
   replacing it. See `docs/decision-records/` for why: the report is itself accountability
   evidence under Art 5(2)/24(1), and overwriting destroys the history that principle expects.
4. Fill `assets/audit-report.md`'s shape: Blockers, then Risks, then Needs-a-human, then a Pass
   summary, then Assumptions & unverified — every finding cites `file:line` and an Article-cited
   obligation.
5. Offer `assets/gap-matrix.csv` and `assets/ropa-template.md` as filled starting drafts based on
   what the audit found, when the scope and findings support it — they aren't mandatory outputs,
   but skipping them when the audit already has the data to populate them wastes what was found.
6. This is the **only** file the audit writes by default. Everything else stays read-only unless
   the user explicitly asks for fixes to be applied.

## Triaging a large codebase

On a large or monorepo codebase, don't attempt exhaustive line-by-line coverage before producing
any findings. Prioritize in this order: (1) the grep recipes from `SKILL.md`'s Phase 1, run
across the whole tree first — they surface the highest-density files fast; (2) any code path
explicitly named "delete account," "export data," "privacy," "consent," or similar — these are
where rights-implementation gaps concentrate; (3) the outbound integrations list (third-party API
clients, config for cloud regions) — usually a short, enumerable list even in a large codebase.
State explicitly in the report which areas were sampled vs. exhaustively reviewed, so a Pass
grade never implies more coverage than was actually done.

## Allowed commands

Read-only inspection only: `grep`/`rg`, `find`, reading files, and cloud CLI `describe`/`get`/
`list` calls to inspect existing infrastructure config (e.g. confirming encryption-at-rest is
enabled on a database, checking a bucket's region). Never run a command that creates, modifies,
or deletes a cloud resource, and never run a mutating `terraform apply`/`aws ... put-*`/
`aws ... create-*` — the audit describes what exists, it doesn't change it. Never attempt to
rewrite git history to remove personal data found in a past commit (see
`references/personal-data-in-code.md` → "Seeds, fixtures, and test data") — name the exposure and
the remediation it needs; don't perform it. The one write action in the entire skill is creating
the report file and, if absent, the `docs/gdpr-audits/` directory itself.
