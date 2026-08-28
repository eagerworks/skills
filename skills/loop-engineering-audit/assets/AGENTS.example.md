<!--
Starter AGENTS.md — copy to the repo root, replace every <placeholder>, delete
anything that doesn't apply, and keep it under ~80 lines. If your tools read
CLAUDE.md instead, create a CLAUDE.md containing just `@AGENTS.md`.
Checklist this satisfies: references/agent-docs.md
-->

# AGENTS.md

## What this is

<Product in one sentence.> <Framework> <version> on <language> <version>, <package manager>. <Monorepo layout: `apps/web` (Next.js), `packages/api` (Rails) — or "single app".>

## Setup

```bash
docker compose up -d db redis      # services
bin/setup                          # installs deps, prepares the DB, copies .env.example → .env
```

No prompts, no secrets needed — `.env.example` values are enough to run everything below.

## Verify — run these before every PR, in this order

| Step | Command | Runtime |
|---|---|---|
| Lint | `bin/rubocop` | ~30 s |
| Typecheck | `pnpm typecheck` | ~1 min |
| Unit tests | `bin/rspec --tag ~type:system` | ~2 min |
| Single spec | `bin/rspec spec/path/file_spec.rb:42` | seconds |
| Full suite (CI runs this) | `bin/rspec` | ~9 min |
| Build | `pnpm build` | ~2 min |

Never run bare `pnpm vitest` or `jest --watch` — they start watch mode and never exit.

## Conventions

- New business logic goes in `app/services/<Verb><Noun>.rb`, one class per file, called from controllers — not in models or controllers.
- Tests: request specs over controller specs; every new service gets a spec asserting behaviour, not just "doesn't raise".
- Migrations: `bin/rails g migration`, never hand-edit `db/schema.rb`.
- Commits: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`). Branch from `main`; name branches `<type>/<short-slug>`.
- Errors: raise domain errors from `app/errors/`; controllers rescue and render, services never rescue broadly.

## Never do this

- `bin/rails db:reset`, `db:drop`, `prisma migrate reset`, or any command that deletes data.
- `bin/deploy`, `kamal deploy`, `pnpm publish`, or anything touching production.
- Edit `vendor/`, `node_modules/`, `db/schema.rb`, `src/generated/`.
- Commit `.env`, `config/master.key`, or any credential. Use placeholders like `your-token` in docs.
- `git push --force` on shared branches.

## Definition of done

- Lint, typecheck, and unit tests green locally.
- New behaviour has a test that asserts it.
- Docs updated when behaviour or commands change (`README.md`, this file, `docs/`).
- PR template filled in; PR targets `main`.
