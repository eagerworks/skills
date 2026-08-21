# PR Review — Configuration

`.eagerworks/pr-review.json`, at the target repo's root, is **entirely optional**. The skill works with no config file at all — it falls back to inferring the base branch and reading whatever `AGENTS.md`/`CLAUDE.md` conventions it finds. Add the file only when a repo needs to override a default or point at repo-specific risk areas.

## Resolution Order

For any given setting: `.eagerworks/pr-review.json` → conventions stated in `AGENTS.md`/`CLAUDE.md` → the skill's built-in default. A later source only fills in what an earlier one didn't set.

## Schema

All fields optional.

```jsonc
{
  // Branch to diff against. Falls back to the repo's default branch (gh repo view) or
  // whichever of main/master/develop exists on origin.
  "baseBranch": "main",

  "review": {
    // Review -> fix -> re-review rounds before the optional fix loop stops and reports
    // (see references/workflow.md). Only relevant when the user explicitly asked for fixes.
    "maxRounds": 3,

    // Repo-specific concerns appended to the four lenses in references/rubric.md, one
    // sentence each. Treat these as known risk areas for this codebase specifically.
    "extraFocus": []
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
    ]
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
    ]
  },
  "localChecks": ["npm test", "npm run lint", "npm run typecheck"]
}
```

See `assets/pr-review.example.json` for a copyable starter combining both.
