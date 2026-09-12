# Audit Open Source Repo — Configuration

`.eagerworks/audit-open-source-repo.json`, at the audited repo's root, is **entirely optional**. The audit works with no config — it infers the audience from evidence, discovers commands from the repo, and writes a dated report under `docs/open-source-audits/`. Add the file to declare the audience explicitly, move the report, or change budgets and samples.

## Resolution order

`.eagerworks/audit-open-source-repo.json` → statements in `CONTRIBUTING.md`/`AGENTS.md`/`CLAUDE.md` (e.g. "we don't require a CLA", "reviews within a week") → the skill's built-in defaults. A later source only fills in what an earlier one didn't set. **Config is input, not evidence** — nothing declared here makes a check 🟢.

## Schema

All fields optional.

```jsonc
{
  // Where the report is written, relative to the repo root. A new dated file
  // every run; never overwritten.
  "reportPath": "docs/open-source-audits",

  // Report language. English by default; never inferred from the conversation's
  // language, so the report is deterministic. Check ids, grade emoji and the
  // P0/P1/P2 labels are never translated.
  "language": "en",

  "project": {
    // "public" (default) | "internal". Decides whether 1.3, 1.8, 3.1 and the
    // public-route half of 3.2 apply, and whether the contribution flow is
    // graded fork-based (2.2) or branch-based. Every check it makes n/a is
    // listed in the footer — never a silent exclusion.
    "audience": "public",

    // "none" (default) | "dco" | "cla". The project's *stance*, not proof of it:
    // declaring "dco" does not make 2.6 🟢 — the audit still looks for both the
    // enforcing check and the documentation, and a mismatch is still a finding.
    // The audit never recommends adopting an agreement (SKILL.md gotcha 7).
    "contributionAgreement": "none",

    // Overrides type detection for the n/a rules: no changelog expectation for an
    // unreleased internal app, no visual-evidence check for a headless library.
    "type": "library"   // library | application | cli | service | monorepo
  },

  // May the audit execute check-only lint/format/typecheck and the test command
  // once each? Never setup, install, migrate, seed, docker, deploy, or publish —
  // those are never run at any setting. false ⇒ 7.7, 8.4 and dry-run stages 6-7
  // become ⚪, never 🟡.
  "runCommands": true,

  // Overrides discovery. Still verified to exist, be non-interactive, and exit
  // correctly — naming a command here does not make it 🟢.
  "commands": {
    "format": "npm run format:check",
    "lint": "npm run lint",
    "typecheck": "npm run typecheck",
    "test": "npm test",
    "check": "make check"          // the aggregate entry point checked by 7.5
  },

  "budgets": {
    "testSeconds": 600,            // 8.4
    "lintSeconds": 120,            // dimension 7 runtime
    "ciMinutes": 20,               // 9.8
    "firstReviewDays": 14,         // 10.2 — a promise stated in CONTRIBUTING
                                    //        always wins over this, and the
                                    //        report says which was used
    "staleOpenPrDays": 90          // 10.3
  },

  "history": {
    "prSample": 20,                // 10.1-10.3
    "issueSample": 20,             // 5.3, 5.8
    "commitSample": 30,            // 2.3
    // Who counts as "internal" for 10.1. Default: CODEOWNERS union the top
    // committers from `git shortlog -sn --no-merges` over 12 months.
    "maintainers": []
  },

  "plan": {
    // Cap on Plan rows. P0 rows are never truncated — if P0s alone exceed the
    // cap, the cap is ignored and the footer says so. Omitted checks still
    // appear in Findings by dimension; the cap changes the Plan, never a grade.
    "maxItems": 25
  },

  // The first-contribution dry run (report section 4). false ⇒ the section is
  // omitted and the footer says `dry run: disabled by config`. It can't change a
  // grade, because it's derived entirely from grades already assigned.
  "dryRun": { "enabled": true },

  "dimensions": {
    "discoverability":      { "enabled": true },
    "contributionContract": { "enabled": true },
    "community":            { "enabled": true },
    "timeToFirstBuild":     { "enabled": true },
    "taskSurface":          { "enabled": true },
    "codeOrientation":      { "enabled": true },
    "executableGuidelines": { "enabled": true },
    "selfVerification":     { "enabled": true },
    "forkSafeCi":           { "enabled": true },
    "reviewAndRelease":     { "enabled": true }
  }
}
```

## Rules

- A disabled dimension appears in the footer as `dimensions disabled by config: taskSurface` and is excluded from the scorecard and verdict.
- `commands.*` entries are still verified (exist, non-interactive, exit code) — config is input, not evidence, exactly as with `project.contributionAgreement`.
- `runCommands: false` never downgrades a check to 🟡; it turns 7.7, 8.4, and dry-run stages 6-7 ⚪ with "set `runCommands: true` or run `<command>` and report the result" as the question.
- `project.audience: "internal"` makes 1.3, 1.8, 3.1, and 3.2's public-route condition n/a — excluded from grading and disclosed in the footer by check id, never silently dropped, never graded 🔴.
- `project.contributionAgreement` declares a stance; it never substitutes for the check. A repo declaring `"dco"` with no enforcing check and no CONTRIBUTING mention is still 2.6 🟡 or 🔴 as the evidence warrants.
- `history.*` sample sizes are targets, not guarantees — when `gh` returns fewer PRs/issues than requested (a young repo) or can't be read at all, the footer states the achieved sample and the affected checks are ⚪, not penalized.
- `plan.maxItems` is a cap on the Plan only; P0 rows are never truncated by it.
- `dryRun.enabled: false` omits report section 4 entirely, disclosed in the footer as `dry run: disabled by config` — the same non-silent-skip rule as a disabled dimension.

Starter: `assets/audit-open-source-repo.example.json`.
