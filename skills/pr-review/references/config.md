# PR Review — Configuration

`.eagerworks/pr-review.json`, at the target repo's root, is **entirely optional**. The skill works with no config file at all — it resolves the base branch from evidence (see `references/base-branch.md`) and reads whatever `AGENTS.md`/`CLAUDE.md` conventions it finds. Add the file only when a repo needs to override a default or point at repo-specific risk areas.

## Resolution Order

For most settings: `.eagerworks/pr-review.json` → conventions stated in `AGENTS.md`/`CLAUDE.md` → the skill's built-in default. A later source only fills in what an earlier one didn't set.

`baseBranch` is the one exception: an open PR's actual base (`gh pr view --json baseRefName`) outranks `baseBranch` when the branch under review already has one. See `references/base-branch.md` for the full ladder.

## Schema

All fields optional.

```jsonc
{
  // Branch to diff against. Overridden by an open PR's actual base when one exists.
  // If neither is set and the fork point is ambiguous, the skill asks rather than
  // guessing — see references/base-branch.md.
  "baseBranch": "main",

  "review": {
    // Review -> fix -> re-review rounds before the optional fix loop stops and reports
    // (see references/workflow.md). Only relevant when the user explicitly asked for fixes.
    "maxRounds": 3,

    // Repo-specific concerns appended to the five lenses in references/rubric.md, one
    // sentence each. Treat these as known risk areas for this codebase specifically.
    "extraFocus": [],

    // Glob patterns excluded from the full-file read and from producing findings —
    // for generated or vendored files (lockfiles, generated clients, snapshots).
    // Matches are always disclosed in the report (see references/output-format.md);
    // this is a skip, never a silent one. A pattern that would exclude a schema
    // migration or auth/scoping code gets named in the report rather than dropped.
    "ignorePaths": [],

    // Lens 5 (references/rubric.md) — flags a diff that makes an existing doc
    // (AGENTS.md/CLAUDE.md, a docs/ page) assert something false, and suggests
    // (never requires) a decision record for an undocumented, non-obvious choice.
    "documentation": {
      // On by default. Set false to turn the whole lens off — no doc-drift
      // findings, no suggestions. A review run this way discloses it in the
      // report rather than silently omitting the section (references/output-format.md).
      "enabled": true,

      // Where a new decision record belongs in this repo. Unset, the skill infers
      // it from the existing layout and proposes docs/decision-records/ when the
      // repo has no documentation home at all.
      "decisionRecordsPath": "docs/decision-records/",

      // Cap on documentation suggestions per review.
      "maxSuggestions": 2
    }
  },

  // Commands this repo uses to verify a fix during the optional fix loop. Run as-is, in order.
  "localChecks": []
}
```

## Example — Rails Studio

```jsonc
{
  "baseBranch": "main",
  "review": {
    "maxRounds": 3,
    "extraFocus": [
      "Every query scoped to an Organization filters through current_user.organization, not just organization_id from params"
    ],
    "ignorePaths": ["db/schema.rb"],
    "documentation": { "enabled": true, "decisionRecordsPath": "docs/adr/" }
  },
  "localChecks": ["bundle exec rspec", "bundle exec rubocop"]
}
```

## Example — Node Studio

```jsonc
{
  "baseBranch": "main",
  "review": {
    "maxRounds": 3,
    "extraFocus": [
      "Every Prisma query on a tenant-scoped model includes orgId from the session, never only from the request body"
    ],
    "ignorePaths": ["package-lock.json", "src/generated/**"],
    "documentation": { "enabled": true, "decisionRecordsPath": "docs/decision-records/" }
  },
  "localChecks": ["npm test", "npm run lint", "npm run typecheck"]
}
```

See `assets/pr-review.example.json` for a copyable starter combining both.
