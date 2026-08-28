# API Review Checklist

Copy this into a PR description or design doc when reviewing a new or existing endpoint. See
`references/review-checklist.md` for the full explanation of each item.

## Scope

- [ ] Listed every path + method this review covers (not just a sample)
- [ ] Checked the existing API's conventions before judging against a house style (see `SKILL.md`)

## Resource modeling

- [ ] Path uses nouns, not verbs
- [ ] Nesting is at most two levels deep
- [ ] IDs are opaque (UUID/ULID) where exposed externally, or protected by strict authorization
- [ ] Non-CRUD actions are modeled as `POST /resource/{id}/action`, not smuggled into `PATCH`

## HTTP semantics

- [ ] Method matches actual safety/idempotency behavior
- [ ] Status codes are specific (`400` vs `422`, `401` vs `403`, `409` vs `422`)
- [ ] `201` responses include `Location`
- [ ] `PUT` replaces the whole resource; partial updates use `PATCH`

## Payloads & errors

- [ ] Key casing is consistent across the whole API
- [ ] Error responses don't leak stack traces, SQL, or file paths
- [ ] Error shape is consistent across endpoints (one envelope, not several)
- [ ] Money is integer minor units, never a float; timestamps are RFC 3339 UTC

## Collections

- [ ] Every collection endpoint has a default *and* max page size
- [ ] Pagination style is safe under concurrent writes, or the trade-off is documented
- [ ] Unknown filter/sort parameters are rejected (422), not silently ignored

## Versioning

- [ ] Breaking changes are behind a version bump
- [ ] Deprecated endpoints/fields carry `Deprecation`/`Sunset` headers and a documented migration path

## Auth & security

- [ ] Every `/{resource}/{id}` handler checks object-level ownership, not just authentication
- [ ] No credentials/tokens in URLs or query strings
- [ ] Request bodies are validated against an explicit writable-fields allowlist (no mass assignment)
- [ ] Rate-limited endpoints return `429` with `Retry-After`/`RateLimit-*` headers

## OpenAPI

- [ ] Spec documents every reachable status code, not just the happy path
- [ ] Spec matches actual behavior (checked against real responses, not assumed)

## Findings

List findings in severity order: security/correctness → breaking-change risk → consistency → polish.
State the concrete failure and file/line, not just the rule violated.
