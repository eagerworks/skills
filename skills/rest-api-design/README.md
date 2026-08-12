# rest-api-design

A portable agent skill for designing and reviewing REST APIs — resource modeling, HTTP semantics,
error shapes, pagination, versioning, auth/security, and OpenAPI. Works with Claude Code, Cursor,
GitHub Copilot, Codex, Amp, and any agentic coding tool that can read markdown files.

## What it covers

- Resource modeling: nouns vs verbs, nesting, identifiers, relationships, non-CRUD actions
- HTTP method semantics, the full status-code catalog, `PUT` vs `PATCH`, `ETag`/idempotency
- Request/response payload conventions and RFC 9457 (`application/problem+json`) error shapes
- Pagination (cursor and offset), filtering, sorting, bulk operations, long-running operations
- Versioning strategy, breaking-vs-additive changes, and a deprecation policy
- Auth mechanisms, object-level authorization, rate limiting, and common security pitfalls (IDOR,
  mass assignment, enumeration)
- Writing and linting an OpenAPI 3.1 spec
- Auditing an existing API and triaging findings by severity
- A survey-first workflow that makes new endpoints match an existing API's conventions instead of
  imposing a house style on top of them

## Layout

```
SKILL.md                          # hub: survey-first step, house style, checklist, gotchas (agent entrypoint)
references/
  resource-modeling.md            # nouns vs verbs, nesting, IDs, relationships, non-CRUD actions
  http-semantics.md               # method safety/idempotency, status codes, PUT vs PATCH, ETag
  payloads-and-errors.md          # body shape, key casing, scalar conventions, RFC 9457 errors
  collections.md                  # pagination, filtering, sorting, bulk ops, long-running operations
  versioning-and-evolution.md     # URL/header versioning, breaking changes, deprecation
  auth-and-security.md            # auth mechanisms, authorization, rate limiting, security pitfalls
  openapi.md                      # OpenAPI 3.1 authoring, $ref reuse, linting, codegen
  review-checklist.md             # auditing an existing API, triaging findings
assets/
  openapi.starter.yaml            # annotated OpenAPI 3.1 skeleton (collection + item + action)
  problem-responses.json          # canonical RFC 9457 payloads per status code
  api-review-checklist.md         # copyable checklist for a design/PR review
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching [`references/`](references/)
file on demand, so the entrypoint stays lean while the full knowledge base is always available.

## Existing-API-first, not house-style-first

The skill states one opinionated default style (plural kebab-case paths, `camelCase` JSON, cursor
pagination, RFC 9457 errors, URL versioning) — but before applying it, `SKILL.md` has the agent survey
the repo for an API that already exists and match its conventions instead. The house style is a
fallback for greenfield work, not something imposed over an established API; deviations are raised as
explicit recommendations rather than applied silently.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill rest-api-design
```
