# Repo Handoff — Configuration

`.eagerworks/repo-handoff.json`, at the analyzed repo's root, is **entirely optional**. The analysis works with no config — it discovers everything from the repo and writes the report to `docs/repo-handoff.md`. Add the file only to move the report, forbid command execution, disable a dimension that genuinely doesn't apply, or pre-declare context the receiving team already has.

## Resolution order

`.eagerworks/repo-handoff.json` → statements in `AGENTS.md`/`CLAUDE.md` → the skill's built-in defaults. A later source only fills in what an earlier one didn't set.

## Schema

All fields optional.

```jsonc
{
  // Where the report is written, relative to the repo root. Overwritten on every run
  // (answers already filled in are carried over).
  "reportPath": "docs/repo-handoff.md",

  // Set to false to forbid running lint/typecheck/test/audit commands. Discovery
  // probes (git, grep, gh reads) always run. Check 3.2 becomes ⚪ when false.
  "runCommands": true,

  // Wall-time budget for the single test run, in seconds.
  "budgets": { "testSeconds": 600 },

  // Receiving-team context that turns ⚪ into 🟢 without asking. Use only for
  // facts you already hold — this is not a place to guess.
  "known": {
    "hosting": "Fly.io org 'your-org', already transferred",
    "ownedServices": ["sentry", "github"]
  },

  // Disable a dimension that does not apply. Disabled dimensions are always
  // disclosed in the report footer.
  "dimensions": {
    "overview": { "enabled": true },
    "environment": { "enabled": true },
    "quality": { "enabled": true },
    "infrastructure": { "enabled": true },
    "data": { "enabled": true },
    "services": { "enabled": true },
    "security": { "enabled": true },
    "codeHealth": { "enabled": true },
    "process": { "enabled": true },
    "operations": { "enabled": true }
  }
}
```

Services listed under `known.ownedServices` still appear in the **Service inventory** table, but their ownership questions (6.2) are graded 🟢 with "declared in config" as evidence. Everything else about them (secret location, rotation, webhooks) is still checked.

## Starter

Copy `assets/repo-handoff.example.json` to `.eagerworks/repo-handoff.json` and delete what you don't need.
