# Design QA Test Cases — Integrations

Pushing the finished plan to a QA tool is optional and configured — never assumed. The invariant, stated once because it governs everything below: **the chat report is delivered in every case — a missing or unauthenticated integration never blocks or fails the run.** Same posture `pr-review` takes with `gh`.

## Detecting What's Actually Reachable

Before offering or attempting a push, confirm the destination is usable — don't try first and explain the failure after:

- **MCP tools** — check the session's available tools (`ListMcpResourcesTool` or the tool list itself) for a matching server (Linear, Notion, an Atlassian/Jira connector).
- **CLIs** — `command -v gh`, `command -v jira`, then confirm auth (`gh auth status`) before treating it as usable.

This check happens both during setup (`references/config.md` — so the skill never offers a destination it can't reach) and at push time (a session's available tools can change between setup and a later run).

## Per-Destination Behavior

| `output.destination.tool` | Preferred path | If unreachable |
| --- | --- | --- |
| `github` | `gh issue comment` (append the plan) or `gh api` for a new sub-issue, per `output.destination.target` | Offer to post once `gh` is installed/authed; print the plan |
| `linear` | Linear MCP — create/update the issue's test-plan section or a linked sub-issue | Offer once the Linear MCP is connected; print the plan |
| `jira` | An Atlassian/Jira MCP, or the `jira` CLI if installed and authenticated | Offer once one is available; print the plan |
| `notion` | Notion MCP — append to the linked page or create a new one | Offer once the Notion MCP is connected; print the plan |
| `testrail` / `xray` / `zephyr` | No direct API integration assumed — render `testCases.format: "table"` (or CSV) for the user to import | Always this path unless a repo wires up its own script via `localChecks`-style tooling — out of scope for this skill |
| `clipboard` | Not a real destination for an agent — treat as `none` and tell the user the plan is ready to copy from the chat | — |
| `none` (default) | — | Chat only; this is the normal, expected configuration for most repos |

## Confirming Before a Push

`output.destination.confirmBeforePush` defaults to `true` because a push is an outward-facing write, unlike everything else this skill does. Before pushing, show exactly what will be written and to where, and wait for confirmation — same posture the harness expects of any action "hard to reverse or outward-facing." Setting it `false` is a deliberate per-repo choice (e.g. a team that always wants the plan filed automatically) and should be treated as durable authorization, not re-confirmed every run.

## What Gets Disclosed

Every one of these is a one-line disclosure in the report, never silent:

```markdown
_Not pushed to Linear — the Linear MCP isn't connected in this session. Install/connect it,
or paste the plan above manually._

_Pushed to LIN-482 as a comment._

_Push skipped — output.destination.tool is "none"._
```
