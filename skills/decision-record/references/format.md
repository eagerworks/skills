# Format

Everything mechanical about a record: where it goes, what it's called, and what it looks like. The rules here are fixed — a record written by this skill has the same shape in every repository.

## Filename

```
docs/decision-records/YYYY-MM-DD--kebab-slug.md
```

- **The date is the date of the decision**, not necessarily today. When backfilling a record for a choice made three weeks ago, use the date it was actually made — the directory listing is a timeline, and a backfilled record dated today puts a 2024 decision after a 2026 one.
- **The slug names the decision, not the component.** It should read as a fragment of the title.

  ```
  ✅ correct
  docs/decision-records/2026-08-21--pessimistic-locking-on-checkout.md
  docs/decision-records/2026-08-28--report-saved-to-a-fixed-path.md

  ❌ wrong
  docs/decision-records/2026-08-21--checkout-2.md          # names the component, and numbers it
  docs/decision-records/2026-08-28--adr-14.md              # names nothing at all
  docs/decision-records/2026-08-28--we-decided-to-use-a-pessimistic-lock.md   # a sentence, not a slug
  ```

- **The filename is the record's identity and its sort order.** Nothing else numbers a record — not the title, not a front-matter field, not a manifest.

The double hyphen between the date and the slug is deliberate: it makes the boundary unambiguous when the slug itself contains hyphens, which it almost always does.

## Title and shape

```markdown
# <One declarative sentence stating the decision>

- **Date:** YYYY-MM-DD

## Context

## Decision

## Consequences

## Related
```

The title is **the decision itself, as a sentence** — not a topic, not a question, not a noun phrase. A reader scanning `ls docs/decision-records/` plus the H1 of each file should learn what was decided without opening anything.

```markdown
✅ correct
# The audit report is saved to a fixed path in docs/, never committed
# Base branch is resolved from evidence, never assumed to be the repo default
# Screenshots are attached through the GitHub CLI, never committed to the repo

❌ wrong
# Report output                                   # a topic
# Where should the report go?                     # a question
# Decision on report path                         # says nothing
# 8. The audit report is saved to a fixed path    # correct sentence, but numbered — see below
```

## Hard rules

### No leading number in the title

A dated filename is already the record's identity, so an in-title sequence number adds nothing and costs something. Two records authored in parallel branches both take the next free number and collide on merge; resolving that collision means editing a title, and every cross-reference that cited the old number is now silently wrong. Dates don't collide the same way, and when two records share a date they still sort deterministically by slug.

### No `Status` field

`Status: Proposed | Accepted | Superseded` is a fixture of the classic ADR format, and it's excluded here because a status is only worth writing if something reads it. In practice nothing does: the field is set once at creation, never updated when reality moves on, and a stale `Accepted` on a record that was quietly abandoned is worse than no field at all.

Supersession is expressed in prose instead. When a later record replaces an earlier one, the **new** record says so in its `## Related` section and the old one stays exactly as written — it is the honest account of what was decided at the time, which is the entire value of keeping it.

### Exactly four `##` sections, in this order

`Context`, `Decision`, `Consequences`, `Related`. Not three, not six, not renamed. No `## Alternatives Considered` section — the alternatives belong inside `Context`, where they can be given the failure mode that made them lose. No `## Notes`, no `## References` — links belong in `Related`.

Sub-headings (`###`) inside a section are fine when a Context genuinely covers two separate judgment calls.

## When the repo already has records

A repo may already keep ADRs in a different convention — numbered `0001-record-architecture-decisions.md` files, a `Status:` field, a `## Alternatives` section, a `docs/adr/` directory.

**Write the format above anyway, and say plainly that you're doing it.** Something like: *"This repo's existing records use numbered filenames and a Status field; I've written this one in the dated, unnumbered shape this skill uses. Tell me if you'd rather I match the existing convention."*

The reasoning is that a predictable output is worth more than local camouflage: an author who wanted their existing convention matched can say so in one sentence and get it, whereas an author who assumed a consistent format and silently got whatever the directory already contained has no way to notice. Never adopt an existing convention silently, and never edit the repo's older records to match this one.

The one thing worth carrying over from an existing directory is its **location**: if records already live in `docs/adr/` or `doc/decisions/`, put the new file there rather than creating a second, competing directory. The path is the only part of the format that follows the repo.
