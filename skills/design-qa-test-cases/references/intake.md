# Design QA Test Cases — Intake

Two independent resolution ladders run before any case is written: what feature is being
tested, and what design (if any) backs its UI. Never guess either — an unresolved rung
falls through to the next one, and the last rung is always "ask" or "mark unavailable",
never "assume."

## Feature Source

1. **A URL was given** (Jira, Linear, GitHub issue/PR, Notion, or similar). Try, in order:
   - A matching MCP tool already available in this session (Linear MCP, Notion MCP, an
     Atlassian/Jira MCP, GitHub via the built-in `gh` tooling).
   - The relevant CLI if installed and authenticated: `gh issue view <N> --json title,body`,
     `gh pr view <N> --json title,body`, `jira issue view <KEY>`.
   - If neither reaches it (no MCP, no CLI, or it's a private link this session can't
     open), **say so and ask the user to paste the ticket's text** — never fabricate
     acceptance criteria from the URL or title alone.
2. **A branch, diff, or PR reference was given** — read the actual diff. This is a
   code-grounded second source of behavior even when a ticket URL is also present; a
   mismatch between the two (the diff does something the ticket doesn't mention, or vice
   versa) is worth a line in the report, not silent reconciliation in either direction.
3. **Plain text describing the feature** — use it as the acceptance criteria directly. If
   it's really one sentence ("add a coupon field to checkout"), that's a thin AC set and
   the resulting plan will be thin too — say so rather than inventing detail to look more
   thorough.
4. **Nothing usable was given** — ask what feature to design cases for. Don't guess from
   the most recent branch name or an open PR unless the user pointed at one.

## Design Source

Resolved independently of the feature source — a feature can have solid ACs and no design
at all, or vice versa.

1. **Figma MCP reachable and a file/frame was named or is inferable** (e.g. linked from
   the ticket) — use it for the cross-cutting UX lens (Lens 8).
2. **A Figma link was pasted directly** — open it the same way.
3. **A screenshot was attached** — treat it as a single-state snapshot: it can inform
   layout/copy cases for that one state, but says nothing about states it doesn't show
   (loading, error, empty) — don't extrapolate those from one image.
4. **An existing, similar screen already in the app** — a reasonable fallback for a design
   system's established patterns (e.g. this app always shows errors as an inline red text
   under the field) — cite the existing screen as the source, don't present it as if it
   were the new design.
5. **None of the above** — say so explicitly in the report and mark every UI/visual case
   under `### ⚪ Not verifiable without more input`. Never invent a layout, a copy string,
   or a responsive breakpoint that wasn't shown anywhere. This is a config-visible choice:
   `design.requireDesignForVisualCases: true` (default) keeps those cases in the ⚪
   section; set `false` only if the repo wants generic, unverified UI cases anyway — and
   the report still marks them as such.

## What Counts as "Reachable"

Before using an integration, confirm it rather than attempting it and reporting a failure
after the fact:

- **MCP tools** — check via `ListMcpResourcesTool` / the tool list actually available in
  this session, not by trying and catching an error.
- **CLIs** — `command -v gh`, `command -v jira`, etc., then confirm auth
  (`gh auth status`) before treating a source as usable.

If a source that looked reachable still fails at fetch time (a private ticket, an expired
session), fall back to the next rung and disclose the fallback in the report — the same
"never silent" rule this skill applies everywhere else.
