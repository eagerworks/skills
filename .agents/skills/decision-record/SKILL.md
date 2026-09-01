---
name: decision-record
description: Write or update a decision record (ADR) in docs/decision-records/ for this repo — eagerworks/skills. Use when a change makes a real judgment call with a defensible alternative that the code alone won't explain (a skill's output contract, where it writes, a naming/config convention, a workflow it deliberately deviates from a precedent on), when the user asks to "record this decision" or "write an ADR", or when landing a non-obvious choice in this collection. Repo-internal tooling — not shipped to skills.sh users.
---

# Decision Record

This repo (`eagerworks/skills`) keeps its architectural decisions as dated markdown
files in `docs/decision-records/`. This skill is the authoritative process for
writing a new one or editing an existing one correctly — filename, title shape,
section structure, and cross-referencing all follow one convention across all
9 existing records, and this skill exists so that convention doesn't have to be
rediscovered (or drift) every time.

## When a record is warranted

Write one when the change is a **choice with a real alternative** that the code
alone won't explain to a future reader — an output contract (where a skill
writes, what its one write is), a naming or config convention, a workflow that
deliberately deviates from an existing precedent, a lens or dimension added to
a rubric, a resolution order for ambiguous input.

Do **not** write one for: a routine edit, a typo/wording fix, anything already
stated plainly in `CLAUDE.md` or `CONTRIBUTING.md`, or a decision with no real
alternative anyone would have picked instead.

## Filename

```
docs/decision-records/YYYY-MM-DD--kebab-slug.md
```

- The date is the date of the decision (not necessarily today, if backfilling).
- The slug names **the decision**, not the skill — e.g. `audit-report-saved-to-docs`,
  not `loop-engineering-audit-2`.
- The filename is the record's identity and its sort order. Nothing else numbers it.

## Title and shape

```markdown
# <One declarative sentence stating the decision>

- **Date:** YYYY-MM-DD

## Context

## Decision

## Consequences

## Related
```

Hard rules, verified against every record currently in this repo:

- **No leading number in the title** (`# 8. ...`). The dated filename is already the
  identity — an in-title sequence number only invites collisions (three records added
  in parallel PRs once collided on `# 8.`) and needs updating for no reason.
- **No `Status` field.** Nothing in this repo reads or has ever set one. A record here
  is not proposed/accepted/superseded — a later record supersedes an earlier one in prose
  (see `## Related`), and the record stays as written.
- Exactly these four `##` sections, in this order, every time.

## Writing each section

- **Context** — name the judgment call(s), and for each one, the failure mode on
  *both* sides of the choice (not just why the rejected option is bad). Cite the
  concrete trigger: a user request quoted, a dogfooding incident, a conflict with
  an existing precedent. Never write a Context that only justifies the answer already
  picked — a reader should be able to see what made this a real decision.
- **Decision** — a numbered, operative list. Each item should be checkable against
  the shipped skill (a rule the report or the workflow actually follows), not a vague
  intention.
- **Consequences** — what a future reviewer or contributor must hold as true because
  of this decision; what it deliberately does *not* do; what a related skill should
  or shouldn't inherit from it.
- **Related** — link sibling records **by relative filename**, never by number:
  `` [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md) ``.
  Also name the `skills/**/references/*.md` (or `SKILL.md`) file whose behavior this
  record justifies, so the operative procedure and its rationale stay linked.

## Other conventions to carry over

- `✅ correct` / `❌ wrong` fenced blocks for contrasts, matching this repo's
  `CONTRIBUTING.md` style.
- Placeholders only, never real secrets or tokens.
- Never invent the "why." If the rationale isn't known, ask the author rather than
  guess — a confidently wrong rationale is worse than no record at all (this is itself
  the reasoning behind `docs/decision-records/2026-08-21--documentation-decision-capture-lens.md`).
- Commit with a `docs:` prefix (Conventional Commits), e.g.
  `docs: record the <slug> decision`.

## Editing an existing record

Only fix factual errors or broken links. Don't rewrite a past record's Context/Decision
to match hindsight — if the decision changed, write a **new** record and link back to
the one it supersedes from its own `## Related` section.

See `template.md` in this skill for a copy-pasteable empty record.
