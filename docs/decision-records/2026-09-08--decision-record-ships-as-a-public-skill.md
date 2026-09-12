# `decision-record` moves from repo-internal tooling to a public skill under `skills/`

- **Date:** 2026-09-08

## Context

`decision-record` was written as `.agents/skills/decision-record/` — hand-authored, not vendored, deliberately excluded from `skills-lock.json`, reached only through the `.claude/skills/decision-record` symlink so Claude Code would auto-discover it inside this repo. Its frontmatter said outright: *"Repo-internal tooling — not shipped to skills.sh users."* Its body opened *"This repo (`eagerworks/skills`) keeps its architectural decisions…"*, cited this repo's own record count ("all 9 existing records" — now stale at 13), and justified its no-leading-number rule with a specific incident here ("three records added in parallel PRs once collided on `# 8.`").

Nothing about the actual guidance — a dated filename, a declarative title, four fixed sections, never inventing the rationale — is specific to this repo. It's a general method for writing an ADR that any repo using this skills collection could use on its own decisions, the same way `commit` or `pr-review` are. Keeping it repo-internal meant every other project pulling skills from this collection had no way to get it.

Two judgment calls followed directly from making it public, each with a real cost on both sides:

1. **Should the shipped skill detect and conform to whatever ADR convention a target repo already has** — numbered files, a `Status` field, an `Alternatives Considered` section — **or should it always write its own fixed shape?** Detecting and conforming keeps the skill from fighting an established local convention, but makes its output nondeterministic: two repos with different existing conventions get differently-shaped records from the same skill, and a repo with a bad existing convention (no rationale requirement, a `Status` field nothing reads) gets that convention perpetuated rather than improved. Always imposing the fixed shape keeps the output predictable and carries forward the specific, load-bearing rules this skill exists to enforce (no number, no `Status`, both-sides Context) — but it means the skill will visibly clash with an existing `0001-*.md`/`Status:` directory the first time it's used there.
2. **Should it take a config file**, the way `commit` (`.eagerworks/commit.json`) and `pr-review` (`.eagerworks/pr-review.json`) do? A config file would let a repo override the format per-project, but the entire value proposition of this skill is that a record looks the same everywhere — a configurable format is a contradiction of the skill's own reason for existing, and it's surface to maintain for a skill whose only output is prose.

## Decision

1. **The skill moved to `skills/decision-record/`** with the full public shape: `SKILL.md` (lean hub), `references/format.md` (filename/title rules, the four sections, why no number and no `Status`), `references/writing.md` (sourcing the "why", section-by-section guide, editing/superseding, the commit), `assets/decision-record.template.md` (moved from the old `template.md`), and `README.md` (human-facing overview matching `commit`'s and `pr-review`'s structure).
2. **`.agents/skills/decision-record/` is deleted.** `.agents/skills/` now holds only the vendored `skill-creator`, which is what `skills-lock.json` actually tracks.
3. **The tracked symlink `.claude/skills/decision-record` now points at `../../skills/decision-record`** instead of `../../.agents/skills/decision-record`, so this repo keeps auto-discovering its own skill from the public location rather than a separate copy.
4. **The format is always imposed, never detected.** `references/format.md` → "When the repo already has records" tells the agent to write the fixed shape regardless of what a target repo's existing ADRs look like, and to say so out loud in the response — never to silently adopt the local convention. The one thing that *does* follow the target repo is the records' *location* (`docs/adr/`, `doc/decisions/`, etc.) if one already exists — only the shape is fixed, not the path.
5. **No config file.** No `.eagerworks/decision-record.json`, no `references/config.md`, no example JSON asset. The only thing repo-specific is the location note in item 4.
6. **All repo-specific language was rewritten as general reasoning** — the stale record count, the `# 8.` collision anecdote, and the specific citation of `2026-08-21--documentation-decision-capture-lens.md` for "never invent the why" were replaced with the underlying reasoning stated generically, so the skill reads the same whether it's running in this repo or any other.
7. **`evals/decision-record/evals.json` was added**, covering: the happy path, a repo with an existing numbered/`Status` convention (write the fixed shape anyway and say so), an unrecoverable missing rationale (ask or `TODO(author):`, never invent), a non-decision (no record warranted), a superseded decision (new record, old one untouched), and the mutation-posture boundary (never push or open a PR).
8. **Root `README.md`'s Available skills table and `CLAUDE.md`'s "The skills" section were updated** to list `decision-record` alongside the other public skills; the old `## decision-record (repo-internal tooling)` section in `CLAUDE.md` was removed.

## Consequences

- Any repo installing this collection (`npx skills add eagerworks/skills --skill decision-record`) can now use this skill for its own ADRs — this is the whole point of the move.
- This repo's own decision records (this one included) are now produced by the same public skill every other project gets, not a private variant — so any future change to the format has to be made once, in `skills/decision-record/`, and this repo's practice inherits it automatically rather than drifting from a duplicate.
- A repo that already keeps ADRs in a different shape (numbered, `Status`-tagged) will see this skill visibly refuse to match that shape on first use. That's deliberate per Decision item 4, not an oversight — but it does mean the skill trades local fit for predictability, and a user who wants the existing convention matched has to say so.
- Because there's no config file, a repo cannot opt into a different shape short of asking in the moment. If real demand emerges for a configurable format, that reopens judgment call 2 above and should get its own record rather than silently adding config to this one.
- `skills/pr-review/references/rubric.md`'s documentation lens still tells a reviewing agent to read a target repo's *existing* ADRs and match their convention when proposing a new one — that instruction was **not** changed by this decision, and it now sits in tension with `decision-record`'s always-impose rule. The two skills don't currently hand off to each other, so nothing breaks today, but a future integration between them will have to resolve that tension explicitly rather than inherit one skill's posture by accident.

## Related

- [2026-06-30--evals-separate-from-skills](2026-06-30--evals-separate-from-skills.md) — why `evals/decision-record/` sits outside `skills/decision-record/` rather than inside it.
- [2026-08-21--documentation-decision-capture-lens](2026-08-21--documentation-decision-capture-lens.md) — the precedent for "never invent the why," carried into `references/writing.md` as general guidance rather than a citation of this specific record.
- `skills/decision-record/SKILL.md` and `skills/decision-record/references/format.md` — the shipped skill this record documents the move of.
- `skills/pr-review/references/rubric.md` (Lens 5) — the still-unreconciled "match the repo's existing convention" instruction noted in Consequences above.
