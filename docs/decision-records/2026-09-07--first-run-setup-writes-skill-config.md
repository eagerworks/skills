# `design-qa-test-cases` writes its own config via a first-run setup, not a hand-copied example

- **Date:** 2026-09-07

## Context

Every existing skill in this collection treats `.eagerworks/<name>.json` as entirely optional and hand-authored: the skill works with zero config, and a repo that wants one copies `assets/<name>.example.json` and edits it. That's the right default when the config only overrides narrow, low-stakes settings (a base branch, an ignore list) that a sensible built-in default handles fine either way.

`design-qa-test-cases` has one setting that default can't paper over: whether, and where, to push a finished plan. QA teams genuinely differ here — Jira, Linear, TestRail, or nothing at all — and the skill has no way to know which without asking. Two problems with just defaulting to "none" and letting the user discover the config file exists:

- **A user is unlikely to find `references/config.md` on their own.** The whole point of progressive disclosure (`CLAUDE.md`) is that reference files load on demand — which means a setting nobody's told the skill to look for stays invisible. A QA team using Linear every day would go several runs never knowing a push option exists. The config itself can stay optional, but awareness of it shouldn't have to be discovered by accident.
- **Asking every run is worse than asking once.** A skill that opens with four setup questions on every invocation is annoying past the first time and trains the user to dismiss the questions rather than answer them thoughtfully.

Also skills are plain markdown with no install hook — there's no `postinstall` script this collection can rely on the way an npm package could. The only place code can run at all is inside a normal skill invocation.

## Decision

1. `design-qa-test-cases` runs a short interactive setup — via `AskUserQuestion`, four questions in one round — the **first time it's invoked in a repo with no `.eagerworks/design-qa-test-cases.json`**, and again whenever the user explicitly asks to (re)configure it. It never asks on any other run.
2. Before offering a push destination, the setup **probes what's actually reachable** (MCP tools in the session, CLIs on `PATH` and authenticated) and only lists options that passed the probe — it never offers Jira as a choice when nothing in the harness can reach it.
3. The setup **writes the config file** with the answers, states what it wrote, and then continues with the run that triggered it — the setup is not a separate step the user has to invoke on its own.
4. This is the skill's only unprompted write, and it's the only place in the plan for `.eagerworks/*.json` files across this collection. `assets/design-qa-test-cases.example.json` still exists, for a repo that wants to check in a config from the start without running the interactive flow (e.g. bootstrapping several repos from a template).
5. A user who declines the setup gets that run's defaults with no file written; the next config-less run offers setup again rather than remembering the decline — the alternative (writing a sentinel to remember "declined") is more persisted state than the choice is worth.

## Consequences

- This is a deliberate deviation from this collection's "config is optional and hand-copied" precedent — a future skill considering the same move should have a similarly concrete reason (a setting genuinely invisible without being surfaced, and genuinely needing to be asked rather than defaulted) rather than copying this pattern by default.
- `pr-review` and `loop-engineering-audit` should **not** retrofit interactive setup from this record — their config settings are override-only and their built-in defaults are sufficient; adding setup there would just add friction with no offsetting benefit.
- The eval suite must cover: first run with no config triggers setup; a run with an existing config never re-asks; a run where nothing probes as reachable skips the push question entirely rather than asking about unreachable tools; and a decline leaves no file behind.
- Every actual push remains additionally gated by `output.destination.confirmBeforePush` (default `true`) — setup only decides the _destination_, not that every future push proceeds without confirmation.

## Related

- [`skills/design-qa-test-cases/references/config.md`](../../skills/design-qa-test-cases/references/config.md) — the setup flow and schema this record justifies.
- [`skills/design-qa-test-cases/references/integrations.md`](../../skills/design-qa-test-cases/references/integrations.md) — the reachability probe and push-confirmation rules.
- [2026-08-28--audit-report-saved-to-docs.md](2026-08-28--audit-report-saved-to-docs.md) — the collection's other precedent for a skill writing beyond its config, for its output artifact rather than its own settings.
- [2026-09-07--create-pr-write-posture.md](2026-09-07--create-pr-write-posture.md) — `create-pr`'s deliberate deviation from the collection's read-only-by-default posture, the closest sibling deviation to this one.
