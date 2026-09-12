# decision-record

A portable agent skill for writing or updating an architecture decision record (ADR) — a dated markdown file under `docs/decision-records/` that captures a judgment call the code alone won't explain: a dated filename, a one-sentence declarative title with no leading number and no `Status` field, and exactly four sections (Context, Decision, Consequences, Related). Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- A fixed filename and title shape (`YYYY-MM-DD--kebab-slug.md`, a declarative sentence for a title) applied the same way in every repo, so a record from one project reads like a record from any other
- No leading number in the title — a dated filename is already the identity, and a sequence number just invites collisions across parallel branches
- No `Status` field — supersession is expressed in prose from a later record's `## Related`, not a field nobody updates once it's set
- Exactly four `##` sections, in order, every time — no `Alternatives Considered`, no `Notes`
- A Context that requires the failure mode on **both** sides of the choice, so a reader can tell the decision was actually hard
- Never invents the "why" — when the rationale isn't in the conversation, the PR, the issue, or the code, it asks the author or leaves an explicit `TODO(author):` rather than fabricating a plausible-sounding reason
- Explicit handling for a repo that already keeps ADRs in a different convention (numbered files, a `Status` field): it says so out loud and still writes this shape, rather than silently blending in
- Editing an existing record is limited to factual fixes; a changed decision gets a **new** record linking back, never a rewrite of the old one

## What it doesn't do

This skill only writes the record. It never touches the code the record is about, never opens a PR, and never pushes — commit the record itself with a plain `docs:` commit (see [`references/writing.md`](references/writing.md) → "Committing"). It also carries no configuration file: the format is the same everywhere on purpose, so there's nothing to override.

## Layout

```
SKILL.md                     # hub: when a record is warranted, the shape, gotchas (agent entrypoint)
references/
  format.md                  # filename/title rules, the four sections, why no number and no Status
  writing.md                 # sourcing the "why", writing each section, editing, committing
assets/
  decision-record.template.md   # copyable empty record with per-section requirements as comments
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

The skill works with zero configuration — the format is fixed by design, so there's nothing to set per repo. The one thing it does take from the target repo is the *location* of an existing records directory (`docs/adr/`, `doc/decisions/`, etc.) if one is already in use — see [`references/format.md`](references/format.md) → "When the repo already has records".

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill decision-record
```
