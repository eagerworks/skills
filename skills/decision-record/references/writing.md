# Writing a Record

The format (`format.md`) tells you what the file looks like. This tells you what to put in it.

## Before writing: find the "why"

A record is only worth the rationale it carries, and the rationale is rarely in the diff. Look, in this order:

1. **The conversation** — what the author just told you, including the alternative they rejected out loud.
2. **The PR description or commit body** — often the only place the reasoning was ever written down.
3. **The issue or ticket** the change closes.
4. **The code itself** — a comment, a test name, a constraint that makes the alternative impossible.

If none of them answer *why this and not that*, **stop and ask the author.** Do not reason backwards from the code to a rationale that sounds plausible; a fabricated "why" is worse than a missing one, because the next reader will cite it as if it were the author's own words and build on it.

When you have most of a record but one specific gap, write the record and mark the gap explicitly rather than papering over it:

```markdown
TODO(author): why a pessimistic lock rather than a Redis-based distributed lock?
The PR explains the move away from optimistic locking but not the choice between these two.
```

A `TODO(author):` line is a legitimate part of a shipped record. A confident invention is not.

## Context

Name the judgment call, and for each one, **the failure mode on both sides of the choice** — not just why the rejected option was bad. A reader should be able to see what made this hard.

Cite the concrete trigger that forced the decision: a request quoted in the author's own words, an incident, a benchmark, a conflict with an existing precedent in the same codebase. A Context that opens "we needed to choose a locking strategy" and closes "so we chose pessimistic locking" has recorded nothing; the reader still doesn't know what optimistic locking cost, or what pessimistic locking costs now.

```markdown
✅ correct
Optimistic retries were thrashing under the checkout-page burst — the seat table saw
40% retry rates at peak, and each retry re-ran the availability query. A pessimistic
lock removes the retry storm but serializes every reservation against the same seat,
which turns a burst into a queue and makes the checkout latency depend on how many
people want the same seat at once. Neither option is free; the question was which
failure we'd rather have at 10x traffic.

❌ wrong
We needed reliable seat reservations. Optimistic locking was causing problems, so we
switched to pessimistic locking, which is more reliable.
```

If the decision covers two genuinely separate calls, give each one its own paragraph or `###` sub-heading. Don't blend them — a reader looking for one shouldn't have to disentangle the other.

## Decision

A **numbered, operative list**. Each item is a rule that can be checked against the thing that shipped, not an intention.

```markdown
✅ correct
1. `SeatReservation#reserve!` takes a row-level lock (`with_lock`) on the seat before
   any availability check. The check inside the lock is authoritative; the one outside
   it is a fast-path filter only.
2. Every other model keeps `lock_version` optimistic locking. This record covers seat
   reservation alone and is not a repo-wide change of strategy.
3. Lock acquisition is capped at 3 seconds; a timeout surfaces as a "seat taken" result
   to the user, never as a 500.

❌ wrong
1. We will use pessimistic locking where appropriate.
2. Care should be taken to avoid long transactions.
```

Bold the operative clause of each item when the list runs long — it makes the section scannable, and it forces you to notice when an item has no operative clause.

Where an item corresponds to a specific file, name it. The record and the code should be reachable from each other.

## Consequences

What a future reviewer or contributor must now hold as true. Three things belong here:

- **What this constrains.** What future work has to respect, and what would now count as a regression.
- **What this deliberately does *not* do.** The scope boundary is the most commonly missed part of a record, and the most useful one — it's what stops a later reader from over-applying the decision.
- **What related work should or shouldn't inherit.** Whether this is a precedent to point at, or a one-off exception that should stay one.

Be honest about the costs the decision accepted. A Consequences section listing only benefits means the Context didn't describe a real choice.

## Related

- **Sibling records, linked by relative filename, never by number:**

  ```markdown
  - [2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md) — the
    resolution-ladder precedent this record follows.
  ```

  Numbers break when files are renamed and mean nothing to a reader browsing the directory. If this record supersedes an earlier one, say so here, in this record — and leave the old one untouched.

- **The file whose behavior this record justifies** — the module, config, workflow doc, or rubric that implements the decision. This is the link that keeps the operative procedure and its rationale from drifting apart:

  ```markdown
  - `app/models/seat_reservation.rb` — the lock this record justifies.
  ```

- **External sources** that actually informed the decision: an RFC, a benchmark, an upstream issue. Not a reading list.

If a record genuinely relates to nothing, keep the heading and write `None.` — an empty section is a claim, and a missing one is an oversight.

## Editing an existing record

Only fix **factual errors and broken links** — a wrong filename, a dead URL, a path that moved, a typo.

Never rewrite a past record's Context or Decision to match hindsight. The record is evidence of what was known and decided at the time; rewriting it destroys the only reason to keep it. If the decision itself changed, write a **new** record and link back to the one it supersedes from the new record's `## Related`.

## Committing

A record is a documentation change — commit it with a `docs:` prefix following [Conventional Commits](https://www.conventionalcommits.org/):

```bash
docs: record the pessimistic-locking-on-checkout decision
```

When the record accompanies the change it describes, it can ride along in that change's PR, but it stays its own commit — a reader running `git log -- docs/decision-records/` should see one commit per record.

Writing the file is this skill's whole mandate. It doesn't push, doesn't open a PR, and doesn't touch the code the record is about.

## Other conventions

- Use `✅ correct` / `❌ wrong` fenced blocks when contrasting a right and wrong approach.
- Placeholders only — `your-token`, `your.domain.com`, `192.168.0.1`. A record explaining an infrastructure or auth decision must not carry real hostnames or credentials into version control.
- Wrap commands, config, and file contents in fenced blocks with a language tag.
