# Design QA Test Cases — Configuration and Setup

Unlike `pr-review` and `loop-engineering-audit`, whose config files are entirely optional and hand-copied from an `assets/*.example.json` if a repo wants one, this skill **writes** `.eagerworks/design-qa-test-cases.json` itself, once, via a short interactive setup — see `docs/decision-records/2026-09-07--first-run-setup-writes-skill-config.md` for why. This file documents both the setup flow and the schema it produces.

## When Setup Runs

- The config file doesn't exist yet in the target repo, **or**
- The user explicitly asks to configure/reconfigure the skill.

Never on any other run — once the file exists, the skill reads it silently like any other config, same as every other skill in this collection.

## The Setup Flow

1. **Probe what's actually reachable** before asking about a push destination — checking first means the question only ever lists real options (see `references/integrations.md` → "Detecting What's Actually Reachable"):
   - MCP tools available in this session (Linear, Notion, Jira/Atlassian, Figma).
   - CLIs on `PATH` and authenticated (`gh auth status`, `jira`).
2. **Ask, using `AskUserQuestion`** (four questions, single round):
   - **Language** — the language test-case plans are written in. Default `en`.
   - **Format** — `steps` (default), `gherkin`, or `table`.
   - **Scope** — manual, automated, or both (default both).
   - **Push destination** — `none` (default) plus only the destinations that probed as reachable in step 1; if nothing probed as reachable, skip this question entirely and set `tool: "none"` without asking.
3. **Write** `.eagerworks/design-qa-test-cases.json` with the answers, filling every other key with its built-in default (see Schema below).
4. **Say what was written** — the resolved config, in one short block — then continue with the run that triggered setup.

**Escape hatches:**

- If the target isn't a writable location (no git repo, read-only filesystem, or the user declines when asked), run this invocation with in-memory defaults and write nothing — say so in one line rather than retrying or erroring.
- Declining setup once doesn't disable it forever — the next run with no config file still offers it, since "no file" and "user said no" aren't distinguishable without writing something, and writing a sentinel just to remember a decline is more state than the decision is worth.

## Resolution Order (once a config exists)

For every setting: `.eagerworks/design-qa-test-cases.json` → conventions stated in `AGENTS.md`/`CLAUDE.md` → the skill's built-in default. A later source only fills in what an earlier one didn't set.

## Report Language

`testCases.language` has its own ladder, same shape as `pr-review`'s:

1. An explicit instruction in the user's request for this run ("write these in Spanish") — wins, once, without touching the config file.
2. `testCases.language` in the config.
3. A stated convention in the target repo's `AGENTS.md`/`CLAUDE.md`.
4. Built-in default: **English.**

**The language the user is chatting in is never an input to this ladder** — see `references/output-format.md` → "Report Language".

## Schema

```jsonc
{
  "testCases": {
    // Language the plan is written in. Set during setup; never inferred from the
    // conversation. See "Report Language" above.
    "language": "en",

    // "steps" (numbered preconditions/steps/expected/data), "gherkin"
    // (Given/When/Then), or "table" (one row per case). references/output-format.md.
    "format": "steps",

    // Which cases to print. Both by default; the skill still reasons about the full
    // set internally so traceability and coverage cross-referencing stay complete —
    // see references/automation.md → "Scope Filtering".
    "scope": ["manual", "automated"],

    "automation": {
      // Test framework to name for automated cases. Unset = inferred from the repo's
      // manifests (references/automation.md). Set this when a repo's convention isn't
      // obvious from a manifest alone (e.g. a custom in-house runner).
      "framework": null,

      // Check existing test files before calling a case a gap; mark it "covered" with
      // a file:line instead of "New" when one already asserts it. Off means every
      // case reports "New" and the report discloses that it didn't check.
      "crossReferenceExistingTests": true
    },

    // Any of the eight lenses in references/techniques.md can be switched off for a
    // repo where it doesn't apply (e.g. a headless service turning off crossCutting).
    // A disabled lens is disclosed in the report, never silently absent.
    "lenses": {
      "crossCutting": true
    },

    // Repo-specific risk areas appended to the eight lenses, one sentence each —
    // same idea as pr-review's review.extraFocus.
    "extraFocus": [],

    // Cap on total cases printed, split proportionally across P0/P1/P2 with P0 never
    // truncated. 0 = uncapped. A truncation is disclosed, never silent.
    "maxCases": 0
  },

  "design": {
    // "auto" (default) uses a reachable Figma MCP when one is available; "off" never
    // attempts it even if reachable, e.g. for a repo that intentionally wants only
    // pasted screenshots reviewed by a human first.
    "figmaMcp": "auto",

    // true (default): a visual/UX case with no design source goes to the ⚪ section
    // instead of being written up unverified. false: allow generic UI cases without a
    // design — still marked as unverified against an actual design in the report.
    "requireDesignForVisualCases": true
  },

  "output": {
    // Path to also save the plan as a file, e.g. "docs/qa/test-cases". null (default)
    // means chat only. See references/output-format.md → "Saving to a File".
    "file": null,

    "destination": {
      // "none" (default) | "github" | "linear" | "jira" | "notion" | "testrail" | "clipboard"
      "tool": "none",

      // "mcp" (default, when available) | "cli"
      "via": "mcp",

      // Project/board/issue id the push targets. Required when tool != "none".
      "target": null,

      // Show what will be pushed and wait for confirmation before every push. Set
      // false only as a deliberate, durable per-repo choice — see
      // references/integrations.md → "Confirming Before a Push".
      "confirmBeforePush": true
    }
  }
}
```

## Example — Manual-Only QA Team, No Push

```jsonc
{
  "testCases": {
    "language": "en",
    "format": "gherkin",
    "scope": ["manual"],
    "extraFocus": ["Every price shown to the user matches the currency of their account, not the org's default"]
  },
  "design": { "figmaMcp": "auto", "requireDesignForVisualCases": true },
  "output": { "file": "docs/qa/test-cases", "destination": { "tool": "none" } }
}
```

## Example — Push to Linear, Table Format for TestRail Import

```jsonc
{
  "testCases": {
    "language": "es",
    "format": "table",
    "scope": ["manual", "automated"],
    "automation": { "framework": "Playwright", "crossReferenceExistingTests": true }
  },
  "output": {
    "destination": { "tool": "linear", "via": "mcp", "target": "ENG", "confirmBeforePush": true }
  }
}
```

See `assets/design-qa-test-cases.example.json` for a copyable starter combining both.
