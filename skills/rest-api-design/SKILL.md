---
name: rest-api-design
description: >
  Expert guide for designing and reviewing REST APIs. Use this skill whenever the user: is adding,
  designing, or renaming an HTTP endpoint or resource; asks what HTTP method or status code to use;
  is shaping a request or response payload, an error format, or a validation-error response; is
  adding pagination, filtering, sorting, or bulk operations to a collection endpoint; is versioning
  an API, planning a breaking change, or deprecating a field/endpoint; is adding authentication,
  authorization, rate limiting, or reviewing an endpoint for security issues (IDOR, mass assignment,
  enumeration); is writing, editing, or linting an OpenAPI/Swagger spec; or is reviewing/auditing an
  existing API for RESTful consistency. Also use when the user says things like "design the API for
  X", "is this endpoint RESTful?", "what status code should I return here", or "review my API" —
  even if they don't say "REST" explicitly.
---

# REST API Design Skill

This skill teaches resource modeling, HTTP semantics, payload/error shapes, pagination, versioning,
auth, and OpenAPI — framework-agnostic (plain HTTP, JSON, and OpenAPI YAML). It states one opinionated
house style with explicit escape hatches, but that style is a *fallback*, not a mandate: **an existing
API's conventions always win.**

## Survey the Existing API First — Do This Before Designing

Before proposing a single path or field name, check whether an API already exists in the repo. Never
design a new endpoint in isolation from the ones that already ship.

```bash
# Find the API surface
find . -iname "openapi*.yml" -o -iname "openapi*.yaml" -o -iname "swagger*.json" -not -path "*/node_modules/*"
grep -rlE "router\.|@(Get|Post|Put|Patch|Delete)Mapping|app\.(get|post|put|patch|delete)\(|resources :" \
  --include="*.rb" --include="*.ts" --include="*.js" --include="*.py" --include="*.java" --include="*.go" .
```

Read a handful of existing endpoints (routes/controllers, serializers, and their tests) and fill in
what you observe:

| Convention | What to check |
|---|---|
| Path casing & pluralization | `/user-accounts` vs `/userAccounts` vs `/users` |
| JSON key casing | `camelCase` vs `snake_case` vs `kebab-case` |
| ID format | UUID/ULID vs sequential integer, in path and in body |
| Pagination style | cursor (`?cursor=`), offset (`?page=`), or none |
| Error envelope | RFC 9457 `problem+json`, a custom `{error: {...}}` shape, or bare messages |
| Versioning scheme | URL (`/v1/`), header, media type, or unversioned |
| Auth mechanism | Bearer JWT, API key header, session cookie, OAuth scopes |
| Timestamp format | RFC 3339 UTC, Unix epoch, locale-formatted |

**The rule: match what exists.** If the codebase's convention conflicts with the house style below,
follow the codebase — do not mix two styles in one API. Call out the conflict explicitly as a
**recommendation for future work**, separate from the endpoint you're actually delivering, so the
inconsistency is a visible decision rather than a silent one.

**No existing API found (greenfield)?** Use the house style below as the default, and say so.

---

## The Default House Style

Use this when there's no existing convention to match. Each row states when to deviate.

| Choice | Default | Deviate when |
|---|---|---|
| Collection paths | Plural, kebab-case nouns: `/user-accounts` | Codebase already uses another casing/plural convention |
| JSON keys | `camelCase` | Codebase or client ecosystem (e.g. Python/Rails) standardizes on `snake_case` |
| IDs | Opaque strings (UUID/ULID) in path and body | Existing schema already keys on sequential integers |
| Pagination | Opaque cursor (`?cursor=&limit=`) | Small, static, rarely-changing collections → offset is fine |
| Errors | RFC 9457 `application/problem+json` | Client ecosystem has an established, incompatible error contract |
| Versioning | URL major version: `/v1/...` | Internal-only API with a single trusted client and lockstep deploys |
| Timestamps | RFC 3339 UTC (`2026-08-12T10:00:00Z`) | Never — this one has no good reason to deviate |
| Money | Integer minor units (`amountCents: 1999`) | Never — floating-point currency is a correctness bug, not a style choice |
| Unsafe POST | Accepts `Idempotency-Key` header | Endpoint is naturally idempotent (e.g. `PUT`) or truly can't be retried |

---

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| Naming resources, nesting, IDs, modeling actions that aren't CRUD | `references/resource-modeling.md` |
| Choosing an HTTP method or status code, `PUT` vs `PATCH`, idempotency, caching | `references/http-semantics.md` |
| Shaping request/response bodies and error payloads | `references/payloads-and-errors.md` |
| Pagination, filtering, sorting, bulk ops, long-running operations | `references/collections.md` |
| Versioning, breaking vs additive changes, deprecation | `references/versioning-and-evolution.md` |
| Auth, authorization, rate limiting, security pitfalls | `references/auth-and-security.md` |
| Writing or linting an OpenAPI 3.1 spec | `references/openapi.md` |
| Auditing an existing API against this skill | `references/review-checklist.md` |

Copyable starter templates live in `assets/`:
- `assets/openapi.starter.yaml` — annotated OpenAPI 3.1 skeleton (collection + item, pagination, errors, auth)
- `assets/problem-responses.json` — canonical RFC 9457 error payloads for common status codes
- `assets/api-review-checklist.md` — copyable checklist for reviewing an endpoint or PR

---

## Designing an Endpoint — Short Checklist

1. Survey the existing API (above) — reuse its conventions before reaching for the house style.
2. Identify the resource(s) and whether this is a collection, an item, a sub-resource, or a
   non-CRUD action on one — see `references/resource-modeling.md`.
3. Pick the HTTP method by semantics (safe? idempotent?), not by habit — see `references/http-semantics.md`.
4. Design the success response: status code, body shape, `Location`/`ETag` headers if relevant.
5. Design the failure responses: which status codes are reachable, and the error body for each —
   see `references/payloads-and-errors.md`.
6. If it returns a collection: add pagination, and decide what's filterable/sortable — see
   `references/collections.md`.
7. Decide if this is a breaking change to something already shipped — see
   `references/versioning-and-evolution.md`.
8. Add authorization at the object level (not just "is authenticated") and check for IDOR/mass
   assignment — see `references/auth-and-security.md`.
9. Write or update the OpenAPI entry — see `references/openapi.md`.

---

## Critical Gotchas

1. **Verbs in paths mean the resource model is wrong.** `/getUsers` or `/users/create` signal you're
   modeling an RPC call, not a resource.
   ```http
   ✅ correct
   POST /orders/{id}/cancel

   ❌ wrong
   POST /cancelOrder?id=123
   ```

2. **Never return `200 OK` with an error in the body.** Clients that check the status code (correctly)
   will treat it as success.
   ```http
   ✅ correct
   HTTP/1.1 404 Not Found
   Content-Type: application/problem+json
   {"type":"about:blank","title":"Not Found","status":404}

   ❌ wrong
   HTTP/1.1 200 OK
   {"success": false, "error": "not found"}
   ```

3. **`PUT` replaces the whole resource; it is not "update some fields."** Partial updates are `PATCH`.
   A `PUT` that silently nulls out omitted fields is a data-loss bug waiting to happen.

4. **`DELETE` and `PUT` must be idempotent.** Calling them twice with the same input must leave the
   resource in the same state as calling them once — no incrementing counters, no duplicate side
   effects on retry.

5. **Offset pagination breaks under concurrent writes.** New inserts shift every subsequent page,
   causing skipped or duplicated items. Default to cursor pagination for anything that mutates.

6. **Don't leak internals in error responses.** No stack traces, SQL fragments, or file paths in a
   4xx/5xx body — that's an information-disclosure bug, not just poor UX.

7. **Every collection endpoint needs a default and a max page size.** An unbounded `GET /items`
   is an outage waiting to happen once the table grows.

8. **A breaking change without a version bump breaks every existing client silently.** Renaming or
   removing a field, tightening validation, or changing a status code are all breaking — see
   `references/versioning-and-evolution.md` for the additive-first alternative.

9. **`404` vs `403` can leak existence.** Returning `403` for a resource that doesn't exist (or `404`
   for one the caller isn't allowed to see) are both legitimate choices — pick one deliberately and
   apply it consistently, don't let it fall out of whatever the ORM raises first.

10. **Sequential integer IDs let attackers enumerate your data.** `/orders/1043` invites scraping every
    order by counting up. Prefer UUIDs/ULIDs for anything exposed externally, or pair sequential IDs
    with strict per-object authorization (never security through obscurity alone).
