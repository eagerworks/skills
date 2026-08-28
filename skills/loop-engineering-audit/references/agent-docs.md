# What a Loop-Ready `AGENTS.md` / `CLAUDE.md` Contains

Dimension 1 grades the project's agent instructions file against this checklist. It's also the outline to recommend when the file is missing (`assets/AGENTS.example.md` is a fillable starter). `AGENTS.md` is the cross-tool name; `CLAUDE.md` is what Claude Code reads — either counts, and a one-line `CLAUDE.md` that says `@AGENTS.md` (or vice versa) is fine.

## Required sections (checks 1.2–1.5)

1. **What this is** — one paragraph: product, framework + version, language version, package manager, monorepo layout if any.
2. **Setup** — the one command (`bin/setup`, `make setup`, `pnpm install && pnpm db:prepare`) and what it needs running first (Postgres via `docker compose up -d db`).
3. **Verify** — the exact non-interactive commands, one per line, in the order an agent should run them, with expected runtime:
   ```markdown
   - Lint: `bin/rubocop` (~30 s)
   - Typecheck: `pnpm typecheck` (~1 min)
   - Unit tests: `bin/rspec --tag ~type:system` (~2 min) · full: `bin/rspec` (~9 min, CI only)
   - Single spec: `bin/rspec spec/path/file_spec.rb:42`
   - Build: `pnpm build`
   ```
4. **Conventions** — where new code goes (services in `app/services`, one class per file), test style (RSpec request specs over controller specs; Vitest with Testing Library), naming, error handling, how migrations are created, commit message format, branch naming and base branch.
5. **Never do this** — the fence list: `db:reset`/`db:drop`, editing `db/schema.rb` by hand, touching `vendor/` or generated dirs, deploy/publish commands, force-pushing, committing `.env`.
6. **Definition of done** — what a PR must include: tests for new behaviour, docs updated, lint/typecheck/tests green, PR template filled.

## Quality bar (checks 1.6–1.7)

- Every command in the file exists and runs. Test it: extract the backticked commands and check each against `bin/`, `package.json#scripts`, `Makefile`, `rake -T`.
- Runtimes are stated so an agent can pick a timeout.
- It says which file is the source of truth when two disagree (e.g. `CLAUDE.md` defers to `AGENTS.md`).
- Kept short — link to `docs/` for depth rather than pasting it. A 400-line file gets skimmed; a 60-line file gets followed.

```markdown
<!-- ✅ correct -->
- Unit tests: `CI=1 pnpm vitest run` (~90 s). Never run bare `pnpm vitest` — it starts watch mode.

<!-- ❌ wrong -->
- Run the tests before opening a PR.
```

## Grading shortcuts

| Situation | Grade |
|---|---|
| No `AGENTS.md`, `CLAUDE.md`, `.cursor/rules/`, or `copilot-instructions.md` | 🔴 1.1 |
| File exists, no verification commands | 🔴 1.3 |
| Verification commands present, one doesn't exist on disk | 🔴 1.6 |
| Commands present and real, but interactive form given (`vitest`, `jest --watch`) | 🟡 1.3 |
| No conventions / no fence list | 🟡 1.4 / 🟡 1.5 |
| Only a `.cursor/rules/` dir with stack notes | 🟡 1.1 — exists, but tool-specific; recommend an `AGENTS.md` other tools also read |
