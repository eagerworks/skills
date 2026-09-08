# design-qa-test-cases

A portable agent skill that turns a feature — a ticket, a diff, or a plain description —
into a rigorous, traceable QA test-case plan by scanning the project's own code:
validations, permission checks, state machines, and existing tests. It designs test cases;
it never writes test code. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and
any agentic coding tool that can read markdown files — and matters most installed inside
the project's own repo, since that's what lets it read the code at all.

## What it covers

- Feature intake from a Jira/Linear/GitHub/Notion ticket, a branch/diff/PR, or plain text —
  never fabricating acceptance criteria from a link it can't actually fetch
- Design intake from a reachable Figma MCP, a pasted Figma link, an attached screenshot, or
  an existing similar screen in the app — and an explicit ⚪ **not verifiable** marker,
  never an invented layout, when none is available
- A code scan per stack (Rails, Node/TypeScript, Python, Go) that pulls exact boundary
  values from validations, the real role × action matrix from policies/middleware, and
  legal/illegal transitions from state machines — instead of guessing them from the ticket
- A coverage cross-reference against the repo's own existing tests, so a case already
  asserted by a passing spec is marked `✅ covered — file:line` instead of re-listed as a gap
- Eight test-design lenses — happy paths, boundaries, negative paths, permissions, state
  transitions, data/lifecycle, integration/async, cross-cutting UX — each case traced to an
  acceptance criterion or a concrete code signal, with a P0/P1/P2 priority
- A manual-vs-automated recommendation per case, with a suggested level (unit/integration/e2e)
  and framework named from what the repo already uses — the skill never writes the test code
- Three output formats (`steps`, `gherkin`, `table`) plus an optional machine-parseable
  `### TEST_CASES` block, and no synthetic `TC-01`-style case IDs — see
  [the decision record](../../docs/decision-records/2026-09-07--test-cases-carry-no-synthetic-ids.md)
- A short interactive setup on first run (or on request) that asks language, format,
  manual/automated scope, and an optional push destination — probing what's actually
  reachable in the harness before offering it — and writes
  `.eagerworks/design-qa-test-cases.json` so it never asks again
- Optional push to a QA tool (GitHub, Linear, Jira, Notion, or a table export for
  TestRail/Xray/Zephyr) when configured, confirmed before every push, with the chat report
  always delivered regardless of whether the destination is reachable

## Layout

```
SKILL.md                          # hub: setup, intake, code scan, the eight lenses, gotchas
references/
  intake.md                       # resolving the feature source and the design source
  code-scan.md                    # what to extract from the repo per stack; coverage cross-ref
  techniques.md                   # the eight lenses in full — the authoritative rubric
  automation.md                   # manual vs. automated, unit/integration/e2e, naming a framework
  output-format.md                # the report shape, the three case formats, the machine block
  integrations.md                 # pushing to a QA tool; graceful degradation
  config.md                       # .eagerworks/design-qa-test-cases.json schema + first-run setup
assets/
  design-qa-test-cases.example.json  # copyable starter config
  test-cases-report.md            # report template, for when output.file is set
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching
[`references/`](references/) file on demand, so the entrypoint stays lean while the full
knowledge base is always available.

## Harness and model

The skill is plain markdown, so it runs in whatever agentic tool loads it. It works with no
external tooling at all — chat-only output, `en`, manual + automated scope. Fetching a
ticket benefits from `gh` (GitHub) or a matching MCP (Linear/Jira/Notion/Figma), and
pushing the plan benefits from the same, but none of it is required — a missing integration
degrades to "print the plan, offer to connect it," never a failure.

**The skill doesn't pick a model.** Whatever model the host agent is already running does
the whole job — there's no model setting in `SKILL.md`, `references/`, or the config file.

## Configuration

The skill writes its own config on first run via a short interactive setup — see
[`references/config.md`](references/config.md) — rather than shipping only a hand-copied
example the way the other skills in this collection do (see
[the decision record](../../docs/decision-records/2026-09-07--first-run-setup-writes-skill-config.md)
for why). To skip that and check in a config from the start, copy
[`assets/design-qa-test-cases.example.json`](assets/design-qa-test-cases.example.json) to
`.eagerworks/design-qa-test-cases.json` and trim it.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill design-qa-test-cases
```
