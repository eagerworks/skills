# `architecture-design` persists its document to `docs/architecture.md` and writes nothing else

- **Date:** 2026-08-29

## Context

`architecture-design` is the first skill in this collection whose output is
a *design*, not a review or an audit. Its deliverable — the v1 architecture of
a system — is produced through a multi-turn interview and is meant to be read
by the whole team, revised as requirements move, and checked against what
actually gets built. That makes it the same kind of artifact as
`loop-engineering-audit`'s report ([2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md)): it outlives the chat
session, it should be diffable across revisions, and its natural home is the
project's own `docs/`.

Two temptations are specific to a design skill and needed a rule:

- An agent that has just designed a system is one step away from
  *scaffolding* it — creating the module directories, writing a
  `docker-compose.yml`, adding dependencies. The user asked for a design.
- The proposed architecture is only as good as the facts behind it, and the
  interview is where those facts come from. A skill that skips ahead to a
  proposal on guessed numbers produces a confident document with invented
  load figures and no open questions.

## Decision

1. The skill **always** writes its full document to `docs/architecture.md`
   at the project's root, creating `docs/` if it doesn't exist. The
   filename is fixed, not dated, so the project carries one current
   architecture doc and `git diff` shows the delta between revisions.
2. The document is **printed in full in chat first**, then saved. The file
   is a copy of the deliverable, not a substitute for it.
3. That file is the skill's **only** write. It is left unstaged — never
   `git add`ed, committed, or pushed — and the skill never scaffolds the
   proposed structure, writes config, or adds dependencies. The document
   describes the system; the team builds it.
4. When `docs/architecture.md` already exists, the skill runs in revise
   mode: existing decision-log rows are kept (superseded, not deleted) and a
   "Changes since last version" section is added, so the decision history
   stays in the document as well as in git.
5. The interview is gated: the skill proposes only once the "enough to
   propose" checklist in `references/interview.md` is met. If the user asks
   to skip ahead, every unmet item is filled with the phase default, marked
   *(assumed)* in the document, and listed under open questions — never
   presented as fact.

## Consequences

- `SKILL.md` gotcha #8 reads "one write, nothing else" like
  `loop-engineering-audit`; reviewers should hold that line — no scaffolding,
  no second file, no auto-commit.
- A repo that `.gitignore`s `docs/` gets told so rather than worked around.
- A document with an empty "Risks & open questions" section is a signal the
  interview was cut short, not a sign of a complete design.

## Related

- [`skills/architecture-design/references/design-workflow.md`](../../skills/architecture-design/references/design-workflow.md) — Phase 4, delivery, and revise mode.
- [`skills/architecture-design/references/interview.md`](../../skills/architecture-design/references/interview.md) — the "enough to propose" gate.
- [`2026-08-28--audit-report-saved-to-docs.md`](2026-08-28--audit-report-saved-to-docs.md) — the precedent this record extends.
