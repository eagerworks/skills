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
  // policy, or a repo whose policy is a single clone, never worktrees).
  // Always disclosed in the report footer — never a silent omission.
  "dimensions": {
    "agentContext":     { "enabled": true },
    "environment":      { "enabled": true },
    "verification":     { "enabled": true },
    "testCoverage":     { "enabled": true },
    "taskDefinition":   { "enabled": true },
    "ciAndGates":       { "enabled": true },
    "guardrails":       { "enabled": true },
    "parallelSessions": { "enabled": true }
  },

  // Advisory loop recommendations (references/loop-catalog.md). They never add a
  // Work Plan row and never move the verdict, so nothing here changes a grade.
  "loops": {
    // On by default. false ⇒ the automation map and the recommended-loops
    // sections are not produced at all, and the footer says
    // `loops: disabled by config` — never a silent omission.
    "enabled": true,

    // Cap on recommended loops. Candidates removed by the cap are disclosed by
    // count in the footer. Set 1–2 when the team wants exactly one next step.
    "maxRecommended": 5,

    // Highest risk level that may be recommended, per the ladder in
    // references/loop-catalog.md: 1 Contained · 2 Shared-state · 3 External-effect.
    // Default 2 — L3 loops reach production and are never proposed by default.
    // Set 1 for an org that will never let a loop touch the default branch.
    // Filtered candidates are disclosed by count, never silently dropped.
    "maxRiskLevel": 2
  }
}
```

## Rules

- A disabled dimension appears in the footer as `dimensions disabled by config: taskDefinition` and is excluded from the scorecard and verdict.
- `commands.*` entries are still verified (exist, non-interactive, exit code) — config is input, not evidence.
- `runCommands: false` never downgrades a check to 🟡; it turns runtime checks ⚪ with "set `runCommands: true` or run `<command>` and report the time" as the question.
- `loops.enabled: false` omits the automation map and the recommended-loops sections and is disclosed in the footer as `loops: disabled by config` — the same non-silent-skip rule as a disabled dimension. It cannot change a grade or the verdict, because loop output never feeds either.
- `loops.maxRecommended` and `loops.maxRiskLevel` are caps, and every candidate they remove is reported by count in the footer.
- `loops.maxRiskLevel: 3` permits an L3 recommendation; it does not make one viable. The prerequisites in `references/loop-catalog.md` still decide — config is input, not evidence, exactly as with `commands.*`.

Starter: `assets/loop-engineering-audit.example.json`.
