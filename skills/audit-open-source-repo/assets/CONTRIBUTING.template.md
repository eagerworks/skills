<!--
Starter CONTRIBUTING.md. Delete every bracketed placeholder and this comment
block once filled in. Point Contributor Readiness Plan rows closing checks
2.1, 2.2, 2.4, 2.5, 2.6, 2.7 at this file — see references/dimensions.md.
-->

# Contributing to [project name]

Thanks for wanting to work on [project name]! This document is the fastest path from an idea to a merged PR.

## Before you write code

- **Bug fix or small change?** Open a PR directly.
- **New feature or breaking change?** Open an issue or a [discussion link] first — this saves you a rewrite if the direction doesn't fit. See [What we accept](#what-we-accept) below.
- **Not sure where to start?** Look for issues labeled [`good first issue`](../../labels/good%20first%20issue) or [`help wanted`](../../labels/help%20wanted).

## Set up your fork

```bash
git clone https://github.com/[your-username]/[repo-name].git
cd [repo-name]
[one bootstrap command — e.g. `make setup`, `npm install`, `bin/setup`]
```

This needs no credentials beyond what's in `.env.example` — if it does, that's a bug, please open an issue.

## Make your change

1. Create a branch: `git checkout -b [prefix]/short-description`
2. [Point at references/dimensions.md check 6.2's "worked path for the common feature kind" — e.g. "Adding a new API endpoint? See docs/architecture.md#adding-endpoints."]
3. Write tests — see [testing conventions link] for where they live and what's expected.
4. Run everything the CI will run, before you push:

   ```bash
   [one aggregate command — e.g. `make check`, `npm run check`]
   ```

## Commit & PR conventions

- Commit messages: [state the convention actually enforced, e.g. Conventional Commits — or say "no fixed format, just be descriptive"]
- Open your PR against `[base branch]`
- Fill in the PR template — link the issue, describe what changed, how you tested it
- [State merge strategy: squash / merge / rebase]

## What we accept

- ✅ Bug fixes, docs, tests, accessibility, performance with a benchmark
- 🤔 New features, new dependencies, breaking changes — discuss first
- ❌ [Name anything genuinely out of scope]

## Review

[Who reviews, how many approvals, response-time expectation — e.g. "A maintainer aims to respond within a week; ping the PR if you haven't heard back after two."]

## Questions

[Where to ask before writing code — Discussions link, chat link, or "open a draft PR and ask there."]

---

By contributing, you agree your changes are licensed under this project's [LICENSE](LICENSE). [Add a DCO/CLA sentence here only if one is actually enforced — see references/dimensions.md check 2.6. Most projects need neither.]
