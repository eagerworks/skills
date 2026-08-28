# Loop Engineering Audit — Configuration

`.eagerworks/loop-engineering-audit.json`, at the audited repo's root, is **entirely optional**. The audit works with no config — it discovers commands from the repo and writes the report to `docs/loop-engineering-audit.md`. Add the file only to move the report, disable a dimension that genuinely doesn't apply, or change the runtime budgets.

## Resolution order

`.eagerworks/loop-engineering-audit.json` → statements in `AGENTS.md`/`CLAUDE.md` (e.g. "the full suite takes 15 min, use `bin/rspec-fast`") → the skill's built-in defaults. A later source only fills in what an earlier one didn't set.

## Schema

All fields optional.

```jsonc
{
  // Where the report is written, relative to the repo root. Overwritten on every run.
  "reportPath": "docs/loop-engineering-audit.md",

  // Verification commands to use instead of what discovery finds. Each must be
  // non-interactive. The audit still verifies they exist and exit correctly —
  // naming a command here does not make it 🟢.
  "commands": {
    "setup": "bin/setup",
    "lint": "bin/rubocop",
    "typecheck": "pnpm typecheck",
    "test": "bin/rspec --tag ~type:system",
    "build": "pnpm build"
  },

  // Wall-time budgets for check 3.5, in seconds.
  "budgets": {
    "testSeconds": 300,
    "lintSeconds": 120,
    "typecheckSeconds": 120,
    "buildSeconds": 600
  },

  // Whether the audit may execute lint/typecheck/test/build commands to measure
  // runtime and exit codes (never setup/migrate/deploy — those are never run).
  // false ⇒ every runtime is ⚪ and commands are graded from source only.
  "runCommands": true,

  // Disable a dimension that doesn't apply (e.g. a library with no CI yet by
  // policy). Always disclosed in the report footer — never a silent omission.
  "dimensions": {
    "agentContext":   { "enabled": true },
    "environment":    { "enabled": true },
    "verification":   { "enabled": true },
    "testCoverage":   { "enabled": true },
    "taskDefinition": { "enabled": true },
    "ciAndGates":     { "enabled": true },
    "guardrails":     { "enabled": true }
  }
}
```

## Rules

- A disabled dimension appears in the footer as `dimensions disabled by config: taskDefinition` and is excluded from the scorecard and verdict.
- `commands.*` entries are still verified (exist, non-interactive, exit code) — config is input, not evidence.
- `runCommands: false` never downgrades a check to 🟡; it turns runtime checks ⚪ with "set `runCommands: true` or run `<command>` and report the time" as the question.

Starter: `assets/loop-engineering-audit.example.json`.
