# Add documentation & decision capture as a fifth `pr-review` lens

- **Date:** 2026-08-21

## Context

`pr-review`'s four lenses — correctness, security & data integrity, repo-convention conformance, test coverage — all ask some form of "is this code right?" None of them asks "will a future reader be able to tell *why* this was written this way?"

That gap is concrete, not hypothetical. A PR routinely commits to one of several defensible approaches — a locking strategy, sync vs. queued work, a new runtime dependency, a tenancy shape, a retry policy, a backfill approach — and the reasoning for picking it lives only in the PR description or the author's head. It never lands in the repo, so it's invisible to whoever opens the file in six months. The same diff can also make an existing document wrong: `references/rubric.md` Lens 3 already treats `AGENTS.md`/`CLAUDE.md` as the authoritative source for repo conventions, and this very skill reads those files before every review — a diff that quietly falsifies one poisons every review (and every human read) that comes after it, and nothing in the skill catches that today.

Four judgment calls had to be made, each with a real failure mode on the other side:

1. **Where does this live in the rubric?** Folding it into Lens 3 (repo-convention conformance) was the cheapest option, but a missing ADR is not a convention violation — Lens 3's own rule is "code that follows a documented convention is never a finding," and an undocumented decision usually *isn't* violating anything written down; there's simply nothing written down yet. Treating it as a convention violation would also block every "not a finding" verdict this lens should be able to give, and would make it easy to conflate "the docs disagree with this diff" (real, Lens 3) with "nothing explains this diff's choice" (a different question).
2. **How does it show up in the report?** A find-and-fix rubric with a hard severity ladder invites treating an undocumented decision as equivalent to a missing test or a scoping bug — but a missing ADR shouldn't block a merge the way a missing auth check does, and folding it into the verdict count risks making `pr-review` read as "your PR isn't done" for something that's a suggestion, not a defect.
3. **How often should it fire?** The core risk with any "should this be documented?" lens is nagging: fire on every PR and the section gets skipped like a linter warning nobody reads, which is worse than not having the lens. But opt-in-only means most repos, which never touch `.eagerworks/pr-review.json`, never get the benefit at all — including repos that already keep a `docs/decision-records/` and would want every qualifying diff checked against it by default.
4. **What does the fix loop do with it?** The `--fix` flow already writes code changes for findings; extending that to *creating* a decision record is tempting, but a decision record's entire value is the author's actual rationale. An agent guessing at "why" from the diff alone and writing it into the repo's permanent decision log is worse than leaving the record unwritten — a confidently wrong "why" gets trusted by later readers and later reviews the same way a correct one would.

## Decision

Add **Lens 5 — Documentation & Decision Capture** to `references/rubric.md`, with two outputs that are deliberately never merged into one:

**A. Doc drift is a normal finding, with a severity.** A diff that makes an already-existing doc assert something false is `high` when the doc is `AGENTS.md`/`CLAUDE.md` (or another the repo treats as authoritative), `minor` otherwise — same evidence bar as any other finding: quote the doc line and the diff line that contradicts it. This resolved judgment call 1 and folds naturally alongside Lens 3 rather than inside it, because it's evaluated with the same rigor as every other finding, not a special case.

**B. An undocumented decision is a suggestion, never a finding.** No severity, never counted in the `**Verdict:**` line, reported in its own `### Documentation` section (markdown) or `### DOCUMENTATION` block (machine-parseable), always subtitled as non-blocking. This resolved judgment call 2.

Judgment call 3 — always on, with three defenses against nagging:
- A high bar: all three of "a real alternative existed," "the rationale is nowhere durable in the repo (a PR/issue/commit description doesn't count)," and "a future reader would ask why and get no answer from the code" must hold. An explicit anti-nag list (routine CRUD, bug fixes, behavior-preserving refactors, dependency bumps, anything already documented) is named directly in the rubric, mirroring Lens 4's existing conservatism language.
- A cap of two suggestions per review (`review.documentation.maxSuggestions`), keeping the longest-lived-consequence ones if more qualify.
- `review.documentation.enabled: false` turns the whole lens off for repos that don't want it — disclosed in the report when set, the same non-silent-skip principle `2026-08-21--ignore-paths-must-be-disclosed.md` established for `ignorePaths`. The default is `true` so a repo gets the benefit without touching config, which was the specific problem with an opt-in design.

Judgment call 4 — under the fix loop, a documentation *suggestion* means drafting the proposed file at its proposed path (matching the format of two existing entries the agent reads first), but every sentence of rationale must be sourced from the PR/issue description, commit messages, code comments, or the diff — anything unsourceable becomes an explicit `TODO(author): why <X> over <Y>?` line rather than a guess. Doc drift (5A) is fixed like any other finding by editing the stale doc.

Silent, opt-in-only, and same-severity-as-code-findings designs were each considered and rejected for the reasons in judgment calls 2–4 above.

## Consequences

- A review against a repo with a genuine documentation gap now surfaces it without the user asking for it by name — at the cost of a `### Documentation` scan running on every review, even ones with nothing to say (mitigated by omitting the section entirely when empty, unlike the severity sections' `_(none)_`, per `references/output-format.md`).
- A repo with `review.documentation.enabled: false` loses both doc-drift findings and suggestions entirely; the report says so rather than silently having an always-empty section.
- The fix loop can now leave a `TODO(author):`-marked draft decision record in a repo's working tree — a new kind of artifact the loop produces beyond code diffs, and one a human still has to finish.
- `assets/pr-review.example.json` and both examples in `references/config.md` model `documentation` with a `decisionRecordsPath`, so a repo copying the starter sees the intended shape.
- This ADR is itself the dogfood: an undocumented decision about how `pr-review` treats undocumented decisions would have been exactly what the new lens flags.

## Related

- [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) — the disclosure precedent (`ignorePaths`) this decision reuses for `review.documentation.enabled: false`.
- [2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md) — the other precedent in this skill for "ask/disclose rather than silently guess."
- `skills/pr-review/references/rubric.md` — Lens 5, the severity-ladder additions, and the suggestion cap this ADR justifies.
- `skills/pr-review/references/config.md` — the `review.documentation` schema entry.
- `skills/pr-review/references/output-format.md` — the `### Documentation` section and the `### DOCUMENTATION` machine block.
