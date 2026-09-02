# Design Workflow

End-to-end procedure. Questions themselves live in `references/interview.md`; design heuristics in `references/decision-guide.md`; the document shape in `references/output-format.md`.

## Phase 0 — Discovery (read-only)

1. Look for a codebase in the working directory: manifests, `README.md`, `AGENTS.md`/`CLAUDE.md`, `docs/`, CI workflows, `Dockerfile`/`docker-compose.yml`, IaC. Record every fact that closes an interview question (language, framework, database, host, CI, tenancy model, existing integrations).
2. Check for `docs/architecture.md`. If present → **Revising an existing doc** below.
3. Parse the user's request into the phase checklist. Mark answered items.
4. Tell the user in two or three lines what you already know and that you'll ask the rest one phase at a time.

Never run setup, install, migrate, or deploy commands during discovery. Reading files is enough.

## Phase 1 — Interview

Run phases 1–7 of `references/interview.md`, one per turn, respecting the four-question cap and the skip rules. After each answer:

- Add facts to the running list.
- Fire any follow-ups whose trigger matched — in the *next* turn, bundled with that phase's questions if there's room, otherwise as their own turn.
- If the answer contradicts an earlier one, ask which is right before moving on.

Stop when the "enough to propose" gate is satisfied.

## Phase 2 — Synthesis

Do this before writing anything the user sees:

1. **Drivers.** From the facts, pick 3–6 architectural drivers, ranked. A driver is a constraint or quality attribute that changes the design if it changes: "2 engineers, 3 months", "tenant data must be isolated", "bookings can never double-book", "must run on customer's Azure". Load numbers and compliance names are drivers; "should be scalable" is not.
2. **Candidates.** Sketch two or three shapes that satisfy the drivers (e.g. modular monolith + Postgres + queue; monolith + separate worker service; serverless functions + managed DB). Use `references/decision-guide.md`.
3. **Pick.** Choose the simplest candidate that meets every driver. Note for each rejected candidate the one driver it fails or the cost it adds — that's the alternatives column of the decision log.
4. **Decisions.** Fill the major-decision list: structure, language/framework, storage, async/background work, auth/tenancy, hosting/deploy, observability, plus any driver-specific ones (search, real-time, files, payments, AI).

## Phase 3 — Proposal & confirmation

Present a **short** proposal in chat — not the document yet:

- The drivers (ranked).
- The shape in five lines: structure, storage, async, hosting, auth.
- The three decisions most likely to be controversial, each with alternatives and the reason.
- Assumptions made where the user accepted defaults.

Ask for objections. Iterate **once**: incorporate changes, restate what changed, then move to delivery. If the user pushes for something the drivers don't justify, state the trade-off in two sentences; if they reaffirm, record it as their decision with their stated reason.

## Phase 4 — Deliver: chat first, then the file

1. Fill `assets/architecture.md` (format: `references/output-format.md`).
2. **Print the complete document in chat** — the whole thing, not a summary.
3. Save the identical markdown:

```bash
mkdir -p docs
# write the document to docs/architecture.md
```

Overwrite an existing file; the project keeps one current architecture doc. Do **not** `git add`, commit, or push it — tell the user it's there and unstaged. If the project's `.gitignore` ignores `docs/`, say so instead of working around it.

Do not create any other file or directory. In particular, do not scaffold the proposed structure, write config files, or add dependencies — the doc describes the system, the team builds it.

## Revising an existing doc

When `docs/architecture.md` already exists:

1. Read it in full. Treat its decision log as settled unless the user says otherwise.
2. Ask what changed — new requirements, a driver that moved, a decision that didn't survive contact with reality. Run only the interview phases those changes touch.
3. Re-run synthesis on the affected decisions. Keep unchanged decisions verbatim.
4. In the document: keep the existing decision-log rows, add new rows for changed decisions (mark the old one "superseded by #N" rather than deleting it), and add a **Changes since last version** section near the top listing what moved and why.
5. Deliver the same way: print in full, overwrite `docs/architecture.md`, leave unstaged.
