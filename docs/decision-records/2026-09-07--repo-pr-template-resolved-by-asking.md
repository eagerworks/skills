# Resolve a conflict between the repo's own PR template and `create-pr`'s description structure by asking, not by a fixed default

- **Date:** 2026-09-07

## Context

A target repo may already have `.github/pull_request_template.md` (or `.github/PULL_REQUEST_TEMPLATE/*.md`). `create-pr` also has its own default description structure (`Summary` / `Problem` / `Solution` / `Screenshots` / `Test plan` / `Checklist`). When both exist, something has to give, and neither silent default is safe.

Always deferring to the repo's template risks nothing on its face, but defeats half the reason someone would invoke this skill: a repo's template can be thin, stale, or simply "add a description," and a user reaching for `create-pr` specifically often wants the more rigorous structure this skill provides. Always overriding the repo's template with the skill's own structure risks the opposite and worse failure: a repo's template may encode a requirement this skill has no way to verify is safe to drop — a compliance sign-off line, a specific reviewer checklist, a security review gate — and silently replacing it could ship a PR that's missing something the team actually depends on.

This repo already has a working precedent for exactly this shape of problem. `pr-review`'s base-branch resolution (`docs/decision-records/2026-08-21--base-branch-resolution.md`) faced the same structure — evidence that doesn't converge on one safe answer — and settled it by asking the user and listing the real candidates, rather than picking a default and hoping it's right. The same reasoning applies here: an existing template and this skill's own structure are two candidates, and the evidence available (the mere existence of a template) doesn't say which one the user actually wants for this PR.

## Decision

1. **When `.github/pull_request_template.md` or `.github/PULL_REQUEST_TEMPLATE/*.md` exists**, the skill asks the user, via `AskUserQuestion`, whether to keep the repo's template as-is or replace it with `create-pr`'s structure for this PR — see `skills/create-pr/references/description.md` → "PR template resolution."
2. **No template found means no question** — the skill's default structure is used without asking, since there's nothing to conflict with.
3. **The question is not re-asked on every run once the answer is made durable**: if `.eagerworks/create-pr.json` sets `pr.sections` explicitly, that config is treated as the recorded answer and the skill uses it without asking again.
4. **Absent that config, every run re-asks.** This is intentional, not an oversight — the absence of `pr.sections` means the repo hasn't made this decision durable yet, and inferring a "sticky" answer from one prior interactive choice would be exactly the kind of silent default this record exists to avoid.

## Consequences

- A repo that wants a fixed, permanent answer sets `pr.sections` once in `.eagerworks/create-pr.json`; until it does, the trade-off is a repeated question rather than a guessed default — a future contributor should not "fix" the repeated question by inferring stickiness from a prior answer, since that reintroduces the silent-default problem this record settles.
- This is the second place in the collection (after `pr-review`'s base-branch ladder) that resolves an ambiguous evidence conflict by asking and naming the real candidates rather than picking one — a future skill facing a similar two-candidates-no-clear-winner situation should look here and at the base-branch record before inventing a third approach.
- `references/description.md` documents this as the operative rule; if the config schema for `pr.sections` changes shape, this record's Decision section should be checked for consistency.

## Related

- [2026-08-21--base-branch-resolution](2026-08-21--base-branch-resolution.md) — the sibling precedent in this repo for resolving ambiguous evidence by asking and listing real candidates, which this decision follows for the same reason.
- `skills/create-pr/references/description.md` → "PR template resolution — ask, never assume" — the operative procedure this record justifies.
