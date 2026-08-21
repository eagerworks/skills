# REST API Design — HTTP Semantics

> Covers choosing a method and status code once you've identified the resource (see
> `resource-modeling.md`).

## Table of Contents

1. [Method semantics](#method-semantics)
2. [Status code catalog](#status-code-catalog)
3. [PUT vs PATCH](#put-vs-patch)
4. [Conditional requests: ETag and If-Match](#conditional-requests-etag-and-if-match)
5. [Location header on creation](#location-header-on-creation)
6. [Idempotency keys for POST](#idempotency-keys-for-post)
7. [Content negotiation](#content-negotiation)
8. [HEAD and OPTIONS](#head-and-options)

---

## Method semantics

Every method has two independent properties: **safe** (doesn't change server state) and
**idempotent** (calling it N times has the same effect as calling it once).

| Method | Safe | Idempotent | Use for |
|---|---|---|---|
| `GET` | Yes | Yes | Read a resource or collection |
| `HEAD` | Yes | Yes | `GET` headers only, no body (existence/metadata checks) |
| `OPTIONS` | Yes | Yes | Discover allowed methods/CORS preflight |
| `POST` | No | No* | Create a resource, or a non-idempotent action |
| `PUT` | No | Yes | Replace a resource entirely (or create at a known ID) |
| `PATCH` | No | No** | Partially update a resource |
| `DELETE` | No | Yes | Remove a resource |

\* `POST` can be made idempotent with an `Idempotency-Key` — see below — but is not by default.
\*\* `PATCH` with JSON Merge Patch is typically idempotent in practice; treat it as not guaranteed
unless you've verified it for your patch semantics.

**Never violate safety or idempotency to save a round trip.** A `GET` that increments a counter, or a
`DELETE` that errors on the second call instead of returning `404`/`204` again, breaks retries, proxies,
and prefetching that all assume these guarantees hold.

## Status code catalog

| Code | Name | Use when |
|---|---|---|
| `200` | OK | Successful `GET`/`PUT`/`PATCH` with a response body |
| `201` | Created | Successful `POST` that created a resource — include `Location` |
| `202` | Accepted | Request accepted for async processing — see `collections.md` for the status-resource pattern |
| `204` | No Content | Successful request with no body (typical for `DELETE`, sometimes `PUT`) |
| `400` | Bad Request | Malformed request the server can't parse or make sense of (bad JSON, wrong type) |
| `401` | Unauthorized | Missing or invalid credentials — *not* about permissions |
| `403` | Forbidden | Authenticated, but not allowed to perform this action on this resource |
| `404` | Not Found | Resource doesn't exist (or the caller shouldn't know it does — see gotcha in `SKILL.md`) |
| `405` | Method Not Allowed | Resource exists, but doesn't support this method |
| `409` | Conflict | Request conflicts with current state (duplicate unique key, concurrent edit, version mismatch) |
| `410` | Gone | Resource existed but was permanently removed (stronger signal than `404`) |
| `422` | Unprocessable Content | Syntactically valid request that fails semantic/business validation |
| `429` | Too Many Requests | Rate limit exceeded — include `Retry-After` / `RateLimit-*` headers |
| `500` | Internal Server Error | Unhandled server-side failure — body must not leak internals |
| `503` | Service Unavailable | Server temporarily can't handle the request (overload, maintenance) |

**`400` vs `422`:** `400` means the server couldn't even parse/understand the request (bad JSON,
wrong content type). `422` means it understood the request perfectly but the *content* fails
validation (a required field is missing, an email is malformed, a date is in the past). Most
"field X is invalid" errors are `422`, not `400`.

**`409` vs `422`:** `409` is about *state* — the request is valid but conflicts with something that
already exists or has changed (duplicate email on signup, optimistic-lock version mismatch). `422` is
about the *request itself* being invalid regardless of current state.

## PUT vs PATCH

`PUT` replaces the entire resource with what's in the body — any field the client omits is treated as
absent/reset, not "leave unchanged." `PATCH` applies a partial change; omitted fields stay untouched.

```http
✅ correct — PATCH only changes what's given
PATCH /invoices/{id}
{ "status": "paid" }
→ every other field on the invoice is untouched

❌ wrong — PUT with a partial body silently blanks the rest
PUT /invoices/{id}
{ "status": "paid" }
→ if the resource has a "notes" field not included here, a naive replace-semantics server nulls it out
```

Two standard `PATCH` body formats:
- **JSON Merge Patch (RFC 7396)** — a partial object merged shallowly into the resource; `null` means
  "delete this field." Simple, good default for flat resources.
- **JSON Patch (RFC 6902)** — an explicit list of `{op, path, value}` operations (`add`/`remove`/
  `replace`/...). More precise for arrays and deeply nested structures, more verbose.

Default to **JSON Merge Patch** unless you need array-element-level operations, then use JSON Patch.
Document which one an endpoint uses — the two are not interchangeable and look similar at a glance.

## Conditional requests: ETag and If-Match

Use `ETag` to let clients detect concurrent modification and cache safely:

```http
GET /invoices/{id}
→ 200 OK
  ETag: "a1b2c3"

PATCH /invoices/{id}
  If-Match: "a1b2c3"
→ 200 OK                          # ETag matched current state, update applied
→ 412 Precondition Failed         # someone else changed it since the client read it
```

This is the standard way to implement optimistic concurrency control over HTTP — prefer it over a
custom `version` field in the body when the client can reasonably keep the `ETag` around.

## Location header on creation

Every `201 Created` response must include a `Location` header pointing at the new resource, in
addition to (usually) returning the created representation in the body:

```http
POST /invoices
→ 201 Created
  Location: /invoices/inv_01H8X...
  Content-Type: application/json

  { "id": "inv_01H8X...", "status": "draft", ... }
```

## Idempotency keys for POST

`POST` is not idempotent by default — a retried request (dropped connection, client timeout) can
create a duplicate resource. For unsafe `POST`s where duplication is costly (payments, orders),
accept an `Idempotency-Key` header and return the original response for a repeated key instead of
creating a second resource:

```http
POST /payments
Idempotency-Key: 7c9e6679-7425-40de-944b-e07fc1f90ae7

{ "amount": 1999, "currency": "usd" }
```

Server behavior: on first sight of a key, process normally and store the key → response mapping (with
an expiry, e.g. 24h). On a repeat with the same key and same body, return the stored response instead
of reprocessing. On a repeat with the same key and a *different* body, return `422` — the client is
misusing the key.

## Content negotiation

Default to `application/json` for both request and response, declared via `Content-Type` and `Accept`.
Reject unsupported types explicitly:

```http
✅ correct
POST /invoices
Content-Type: application/json
Accept: application/json

❌ wrong — silently accepting anything and guessing the format
```

If a request has an `Accept` header the server can't satisfy, return `406 Not Acceptable` rather than
guessing. If a request has a `Content-Type` the server can't parse, return `415 Unsupported Media Type`.

## HEAD and OPTIONS

- `HEAD` — implement it wherever `GET` exists; it must return the same headers `GET` would, no body.
  Useful for existence checks and content-length probes without transferring the payload.
- `OPTIONS` — used by browsers for CORS preflight; return `Allow`/`Access-Control-Allow-*` headers.
  Most frameworks handle this automatically — verify it isn't disabled rather than hand-rolling it.
