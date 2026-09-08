---
name: decision-record
description: >-
  Writes or updates an architecture decision record (ADR) as a dated markdown file under
  docs/decision-records/ — a one-sentence declarative title with no leading number and no
  Status field, and the four sections Context, Decision, Consequences, Related. Use when a
  change makes a real judgment call with a defensible alternative that the code alone won't
  explain, when asked to "record this decision", "write an ADR", or "document why we chose
  this", or when a review flags an undocumented decision. Never invents a rationale — it asks
  when the "why" isn't known.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Decision Record Skill

Turns a judgment call that's still only in someone's head — or in a PR description that will scroll out of sight — into a durable markdown file the next reader can find. The record captures **why** a choice was made and what the real alternative was, so the reasoning survives the person who had it.

The format is fixed, not negotiated: a dated filename, a declarative one-sentence title with no leading number, no `Status` field, and exactly four sections. That's the whole point — a record written by this skill looks the same in every repo, so a reader who has seen one can read them all. See `references/format.md` for each rule and why it's that way, including what to do when a repo already keeps records in a different shape.

**Mutation posture.** Writing the file is the point of this skill, so creating or editing a record under `docs/decision-records/` doesn't need a separate confirmation once the user has asked for one. The mandate stops there: it never pushes, opens a PR, or edits the code the record is about. Committing the record is a plain `docs:` commit — see `references/writing.md`.

## When a Record Is Warranted

Write one when the change is a **choice with a real alternative** that the code alone won't explain to a future reader:

- An output contract — where something writes, what its one write is, what it deliberately never touches.
- A naming, layout, or configuration convention that other work will have to follow.
- A workflow that deliberately deviates from an existing precedent in the same codebase.
- A lens, dimension, or rule added to a rubric or checklist.
- A resolution order for ambiguous input (which signal wins when two disagree).
- A dependency, protocol, or storage choice picked over a named competitor.

Do **not** write one for: a routine edit, a typo or wording fix, anything already stated plainly in the repo's agent instructions or contributing guide, or a decision with no real alternative anyone would have picked instead. A record that documents a non-choice trains readers to skip the directory.

If you're unsure, the test is: *could a competent contributor six months from now reasonably undo this, not knowing what you knew?* If yes, write it.

## The Shape at a Glance

```
docs/decision-records/YYYY-MM-DD--kebab-slug.md
```

```markdown
# <One declarative sentence stating the decision>

- **Date:** YYYY-MM-DD

## Context

## Decision

## Consequences

## Related
```

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| Filename and slug rules, the title shape, the four sections, why no number and no `Status` | `references/format.md` |
| What goes in each section, sourcing the "why", editing and superseding, the commit | `references/writing.md` |

Copyable templates live in `assets/`:
- `assets/decision-record.template.md` — an empty record with each section's requirements as comments

## Critical Gotchas

1. **Never invent the "why."** If the rationale isn't in the conversation, the PR description, the issue, or the code, ask the author — or leave a `TODO(author):` line naming exactly what's missing. A confidently wrong rationale is worse than no record at all, because it gets cited.
2. **A changed decision is a new record, never a rewrite.** Editing a past record to match hindsight destroys the only evidence of what was actually known at the time. Write a new one and link back from its `## Related`.
3. **No leading number in the title, and no `Status` field.** Both are common in other ADR conventions and both are excluded here for concrete reasons — `references/format.md`.
4. **Context must show the decision was real.** State the failure mode on *both* sides of the choice. A Context that only justifies the option already picked is marketing, not a record.
5. **Link siblings by relative filename, never by number.** `[2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md)`, not "see ADR 4".
6. **Don't write a record for a non-choice.** Say plainly that none is warranted and why, rather than producing one to satisfy the request.
7. **Placeholders only.** A record explaining an infrastructure or auth decision must not carry real hostnames, tokens, or credentials into version control.
8. **The repo's existing records don't override this format.** If they use a different shape, say so out loud and still write the shape above — see `references/format.md` → "When the repo already has records".
