# Make the `pr-review` report language a configured setting, never inferred from the conversation

- **Date:** 2026-09-01

## Context

Nothing in `skills/pr-review/` ever named a report language — a grep across `SKILL.md`, every `references/*.md`, and `evals/pr-review/evals.json` for `language|idioma|english|spanish|locale` returned zero hits. Every template in `references/output-format.md` is written in English, but no instruction told the agent to *write* in English; the language was decided implicitly by whatever language the model happened to be conversing in.

That implicit behavior stops being harmless the moment [2026-09-01--inline-pr-review-comments](2026-09-01--inline-pr-review-comments.md) made posting to the PR the default. The report is no longer a private answer to whoever asked for the review — it's a team-facing artifact that lands as inline comments and a review summary on a shared PR. A developer chatting in Spanish would silently post a Spanish review onto a repo whose codebase, commit history, and other reviewers' comments are in English (or vice versa), and the same PR reviewed by two different teammates in two different chat languages would read as if two different tools had reviewed it. The output needed to be deterministic per repo, independent of who happened to type the request and in what language.

Three questions needed settling:

1. **One setting or two?** A repo could plausibly want the console report in one language (whatever the developer reads) and the PR comment in another (whatever the team reads). But `references/output-format.md` already states the console report and the posted comment must be byte-identical in every other respect (`"Same prose the console report uses for that finding"`, `"the comment must read exactly like the console output"`) — letting language be the one place they diverge would break that identity for no benefit a repo has actually asked for, and would double the surface a repo has to configure and a reader has to reason about.
2. **What's the default when nothing is configured?** Falling back to "whatever language the conversation is in" is exactly the status quo this change exists to remove — it reproduces the non-determinism, just with a config key that happens to do nothing until set. English was chosen as the fixed, unconditional default: predictable, matches every existing template and eval string, and is the de facto standard for PR review comments in the ecosystems this skill targets (Rails, Node/TypeScript).
3. **Does the machine-parseable block translate too?** `references/output-format.md`'s `### FINDINGS`/`### DOCUMENTATION` blocks exist so "another skill, script, or agent" can parse them mechanically, keyed on fixed field names (`severity`, `file`, `claim`, `failure`, `fix`, …) and a fixed `critical | high | minor` enum. Translating those would silently break every consumer of that contract the day a repo sets a non-English `review.language` — a regression with no upside, since the block is explicitly "for scripts, not for people reading the PR" and no one asked to localize a machine contract.

## Decision

1. **One setting: `review.language`** in `.eagerworks/pr-review.json`, governing the console report, the inline comment bodies, the review summary body, and every disclosure line as a single unit. A BCP-47 tag or a plain language name (`"en"`, `"es"`, `"pt-BR"`, `"Spanish"`).
2. **Resolution ladder, in `references/config.md`:** (0) an explicit instruction in the user's request, for that run only, never touching the config file; (1) `review.language`; (2) a stated convention in the target repo's `AGENTS.md`/`CLAUDE.md`; (3) built-in default, **English**. Rung 0 exists so a one-off "review this in Spanish" request works without editing config, exactly like other per-run overrides this skill already tolerates (e.g. naming a base branch inline outranks stored config for that run).
3. **The conversation's language is never an input to the ladder.** This is the rule the whole change exists to state explicitly — without it, "no config found" would fall through to the old implicit behavior by omission. An unrecognized or unconfident value falls back to English, disclosed in one line, following the same never-silent-skip precedent as [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md).
4. **Full translation of prose, with one machine-contract carve-out.** Headings, verdict sentences, `**Failure:**`/`**Fix:**`-style labels, captions, and all four disclosure lines follow `review.language`. The `### FINDINGS`/`### DOCUMENTATION` sentinels, field keys, and the `critical | high | minor` enum are never translated — only the `claim`/`failure`/`fix`/`subject`/`where`/`why`/`rationale` prose values inside them do. `references/output-format.md` now carries a worked Spanish example specifically to demonstrate this split, not just assert it.
5. **This governs the report artifact, not the surrounding conversation.** The agent still replies to the user in whatever language the user is using — `review.language` only decides the language of what gets written into the report and posted to GitHub.

## Consequences

- A repo's PR reviews read in one consistent, team-chosen language no matter which teammate — or which language they happen to type in — triggers the review.
- With no config, behavior is now a hard guarantee (English) rather than an accident of the conversation, closing the gap the initial bug report identified: reviewing in Spanish must not silently post a Spanish review to an English-language repo.
- A repo that wants Spanish (or any other language) sets `review.language` once; `assets/pr-review.example.json` and the Rails worked example in `references/config.md` both demonstrate a non-default value so the option is discoverable, not just documented.
- Scripts and the optional fix loop that parse the `### FINDINGS`/`### DOCUMENTATION` block are unaffected by any `review.language` value — the contract's shape never changes, only the prose inside three of its fields.
- `SKILL.md` gains one new Critical Gotcha and one sentence in its posting paragraph rather than any rubric duplication, keeping the progressive-disclosure rule in `CLAUDE.md` intact — full detail lives in `references/config.md` and `references/output-format.md`.

## Related

- [2026-09-01--inline-pr-review-comments](2026-09-01--inline-pr-review-comments.md) — defines the exact surfaces (inline comment body, review summary body) this record now makes language-configurable.
- [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) — the disclose-never-silently-skip precedent the unrecognized-language fallback follows.
- `skills/pr-review/references/config.md` → "Report Language" — the operative resolution ladder this ADR justifies.
- `skills/pr-review/references/output-format.md` → "Report Language" — the translated/never-translated split and the worked Spanish example.
