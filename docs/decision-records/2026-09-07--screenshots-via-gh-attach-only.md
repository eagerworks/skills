# Attach PR screenshots only via the GitHub CLI's documented `--attach` flag, never the undocumented upload endpoint

- **Date:** 2026-09-07

## Context

Getting a screenshot into a pull request body via automation has never had an officially supported path. GitHub's REST and GraphQL APIs have never accepted a direct image upload for an issue or PR body — the web UI's drag-and-drop goes through an internal, undocumented asset pipeline. Every workaround guide for this problem points at the same two options: hit `https://uploads.github.com/user-attachments/assets` directly with a bearer token (it works, and is equivalent to the browser's drag-and-drop, but is unversioned, undocumented, and carries no stability guarantee), or host the image externally (S3, a GitHub Release asset, a base64 data URI) and link to it, which is heavier and only needed at all because the direct path didn't exist.

While researching how `skills/create-pr` should handle screenshots (2026-09-07), it turned out this has changed: GitHub CLI `2.99.0`, GA'd 2026-09-01, added an official, documented `--attach` flag on `gh pr create`, `gh pr edit`, `gh pr comment` (and the `gh issue` equivalents) that uploads a local image or video and rewrites its reference in the PR body in place — exactly the capability the undocumented endpoint and the external-hosting workarounds existed to approximate, now shipped as a first-class, versioned CLI feature. Separately, the official `github/github-mcp-server` was confirmed to have no attachment-upload capability at all ([github/github-mcp-server#738](https://github.com/github/github-mcp-server/issues/738), open, blocked on GitHub not exposing a public upload API) — so an MCP-based integration can't be the answer either, at least not yet.

The judgment call was whether to document the undocumented endpoint anyway, as a fallback for the case where `--attach` isn't available (an older `gh`, or a GitHub Enterprise Server instance the flag doesn't yet support) — it does still work, and a fallback would make the skill more broadly capable. The failure mode of including it: a repo-authored skill in this collection would be steering users toward an unsupported internal API that could disappear or start rejecting agent traffic without notice, for a problem that — for the overwhelming majority of users on a recent `gh` against github.com — is now solved properly. The failure mode of excluding it: a user on an old `gh` or on GHES gets a hard stop instead of a working (if fragile) screenshot upload.

## Decision

1. **`gh --attach` is the only screenshot/video upload path this skill documents or uses**, on `gh` ≥ 2.99.0 — see `skills/create-pr/references/screenshots.md`.
2. **The undocumented `uploads.github.com/user-attachments/assets` endpoint is explicitly excluded**, not merely omitted — `references/screenshots.md` names it and states why it's not used, so a future contributor tempted to "fix" a screenshot gap by reaching for it finds the reasoning already recorded rather than rediscovering it.
3. **`gh` below 2.99.0 is a hard stop with an upgrade instruction** (`brew upgrade gh` or the platform equivalent), not a silent fallback to the undocumented endpoint or to external hosting.
4. **No image available at all is a placeholder plus a question to the user**, never a fabricated URL — consistent with this repo's existing rule that an unsourceable "why" becomes an explicit `TODO(author):` line rather than a guess (`docs/decision-records/2026-08-21--documentation-decision-capture-lens.md`, on the fix loop's decision-record drafting).
5. **External hosting is not a first-resort fallback.** It's mentioned only as an option the user can explicitly ask for (e.g. a file exceeding `--attach`'s size limit), not something the skill reaches for on its own.

## Consequences

- This skill's screenshot capability is bounded by `gh`'s installed version and by GitHub's stated constraints (10 MB images/GIFs, 10/100 MB video, write access required, not supported on GitHub Enterprise Server). A user on GHES or an unupgradable `gh` gets a clear stop, not a degraded workaround.
- If GitHub ever deprecates `--attach`, changes its constraints, or extends it to GHES, that is a new fact requiring a new decision (or an update to this one) — not something a contributor should quietly work around by reintroducing the undocumented endpoint this record explicitly rejected.
- `skills/create-pr/README.md` and `references/screenshots.md` both state this reasoning so the choice is visible to anyone reading the skill, not just to whoever wrote it.

## Related

- [2026-08-21--documentation-decision-capture-lens](2026-08-21--documentation-decision-capture-lens.md) — the repo's existing never-fabricate precedent (an unsourced "why" becomes a `TODO(author):`, never a guess), which this record applies to unavailable screenshots (a placeholder, never a fabricated URL).
- `skills/create-pr/references/screenshots.md` — the operative procedure, version gate, and "what not to do" list this record justifies.
