# pr-review

A portable agent skill for reviewing a diff — a branch, a PR, staged changes, or working-tree changes — against a fixed four-lens rubric: correctness, security & data integrity, repo-convention conformance, and test coverage. Built for Ruby on Rails and Node/TypeScript codebases. Works with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Scope detection: full branch vs. base, staged changes, working tree, or a specific GitHub PR
- The four review lenses with Rails and Node/TypeScript examples for each
- A conservative severity ladder (critical / high / minor / not a finding) that avoids padding the list with style nits
- Reviewing against an issue or PR's acceptance criteria, with a test-coverage check that requires the test to actually assert the behavior, not just exercise the code path
- Two output formats: a readable markdown report by default, and a machine-parseable `### FINDINGS` block for automated consumers
- An optional review → fix → re-review loop, off by default — the standard review never edits, commits, or pushes
- Optional per-repo configuration for the base branch, local verification commands, and repo-specific risk areas (`extraFocus`)

## Layout

```
SKILL.md                        # hub: scope detection, the four lenses, severity, gotchas (agent entrypoint)
references/
  rubric.md                     # full four-lens checklist, severity ladder, Rails + Node examples
  workflow.md                   # end-to-end review flow, issue/PR review, optional fix loop
  output-format.md              # markdown report format + the ### FINDINGS block
  config.md                     # .eagerworks/pr-review.json schema and resolution order
assets/
  pr-review.example.json        # copyable starter config (Rails + Node examples)
  code-reviewer.agent.md        # optional Claude Code subagent wrapper around this skill
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/) file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Configuration

The skill works with zero configuration — it infers the base branch and reads whatever `AGENTS.md`/`CLAUDE.md` conventions exist in the target repo. To override the base branch, point at a repo-specific risk area, or wire up local verification commands for the optional fix loop, add `.eagerworks/pr-review.json`. See [`references/config.md`](references/config.md) for the full schema and [`assets/pr-review.example.json`](assets/pr-review.example.json) for a starter.

## Claude Code subagent (optional)

[`assets/code-reviewer.agent.md`](assets/code-reviewer.agent.md) is a copyable Claude Code subagent definition that wraps this skill's rubric in a dedicated, read-only reviewer — copy it to `.claude/agents/code-reviewer.md` in your project if you want the review to run as an isolated subagent call. It's optional: the skill works standalone in any agentic tool without it. The subagent points back at this skill's own `references/rubric.md`, `workflow.md`, and `output-format.md`, so the `pr-review` skill itself must also be installed in the same project (e.g. `.claude/skills/pr-review/`) for those references to resolve — copying only the subagent file on its own leaves those paths unresolvable.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill pr-review
```
