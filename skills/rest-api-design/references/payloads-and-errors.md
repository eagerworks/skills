# REST API Design — Payloads & Errors

> Covers the shape of request/response bodies and error responses. For which status code accompanies
> an error, see `http-semantics.md`.

## Table of Contents

1. [Response body shape](#response-body-shape)
2. [Key casing and field conventions](#key-casing-and-field-conventions)
3. [Null vs omitted](#null-vs-omitted)
4. [Enums, dates, money, and other scalar conventions](#enums-dates-money-and-other-scalar-conventions)
5. [The default error shape: RFC 9457](#the-default-error-shape-rfc-9457)
6. [Field-level validation errors](#field-level-validation-errors)
7. [Error catalog by status code](#error-catalog-by-status-code)

---

## Response body shape

Default to returning the resource **bare** (not wrapped in a `data` envelope) for single-item
responses, and a small envelope only for collections, where metadata (pagination) needs somewhere to
live:

```json
✅ correct — single item, bare
GET /invoices/{id}
{ "id": "inv_01H...", "status": "paid", "amountCents": 1999 }

✅ correct — collection, minimal envelope for pagination metadata
GET /invoices
{
  "data": [ { "id": "inv_01H...", ... }, ... ],
  "pagination": { "nextCursor": "eyJpZCI6...", "hasMore": true }
}
```

```json
❌ wrong — wrapping single items for no reason, forces every client to unwrap
GET /invoices/{id}
{ "data": { "id": "inv_01H...", ... }, "meta": {} }
```

If the codebase already wraps single items (some frameworks default to this), match it — this is a
house-style default, not a hard rule (see `SKILL.md` → Survey the Existing API First).

## Key casing and field conventions

Pick one casing and hold it across every endpoint: `camelCase` is the house-style default for JSON
(matches JavaScript/JSON idiom), but `snake_case` is equally legitimate and often already the norm in
Python/Ruby-backed APIs. What's not acceptable is mixing them:

```json
❌ wrong — inconsistent within the same object
{ "invoiceId": "inv_01H...", "amount_cents": 1999, "Status": "paid" }
```

## Null vs omitted

Decide deliberately what a missing value means, and state it in the OpenAPI spec:

- **`null`** — the field is a known part of the resource, and its value is explicitly absent
  (e.g. `"cancelledAt": null` means "not cancelled").
- **omitted** — the field doesn't apply in this context at all (e.g. a `PATCH` body only includes
  fields being changed under JSON Merge Patch semantics).

Don't use `null` and omission interchangeably for the same field — a client parsing the response needs
to know which one it's dealing with.

## Enums, dates, money, and other scalar conventions

| Type | Convention | Example |
|---|---|---|
| Enums | Lowercase or SCREAMING_SNAKE string, never a bare integer | `"status": "pending_review"` not `"status": 2` |
| Timestamps | RFC 3339, UTC, with `Z` suffix | `"createdAt": "2026-08-12T10:00:00Z"` |
| Dates (no time) | ISO 8601 date only | `"dueDate": "2026-09-01"` |
| Money | Integer minor units + explicit currency, never a float | `"amountCents": 1999, "currency": "usd"` |
| IDs | String, even if internally numeric | `"id": "inv_01H8X3Z..."` |
| Durations | ISO 8601 duration or explicit unit in the field name | `"timeoutSeconds": 30` |

Floats for money are a recurring, entirely avoidable class of bug (rounding errors compound across
a ledger) — treat this one as non-negotiable, not a style preference.

## The default error shape: RFC 9457

Use [RFC 9457 Problem Details](https://www.rfc-editor.org/rfc/rfc9457) (`application/problem+json`)
as the default error body for every 4xx/5xx response:

```http
HTTP/1.1 404 Not Found
Content-Type: application/problem+json

{
  "type": "https://api.example.com/errors/invoice-not-found",
  "title": "Invoice Not Found",
  "status": 404,
  "detail": "No invoice exists with id inv_01H8X3Z.",
  "instance": "/invoices/inv_01H8X3Z"
}
```

Core members: `type` (a URI identifying the error kind — `about:blank` is fine if you don't maintain a
docs page per error type), `title` (short, stable, human-readable summary of the type), `status`
(repeats the HTTP status for convenience), `detail` (specific to this occurrence), `instance` (the
request path, useful for support/debugging). Extend with custom members (e.g. `errors[]` below) as
needed — RFC 9457 explicitly allows extensions.

If the codebase already has an established custom error shape, match it (see `SKILL.md`) and note RFC
9457 as a recommendation rather than retrofitting every existing endpoint.

## Field-level validation errors

For `422` responses from body validation, extend the problem body with a field-level `errors` array so
clients can map failures back to form fields:

```json
{
  "type": "https://api.example.com/errors/validation-failed",
  "title": "Validation Failed",
  "status": 422,
  "detail": "The request body failed validation.",
  "errors": [
    { "field": "email", "message": "must be a valid email address" },
    { "field": "amountCents", "message": "must be a positive integer" }
  ]
}
```

Use a stable `field` path (dot-notation for nested fields, e.g. `billingAddress.postalCode`) so
clients can programmatically highlight the right input — not just a human-readable sentence.

## Error catalog by status code

Design every endpoint's *reachable* error responses up front, not just the happy path — an OpenAPI
spec with only a `200` documented is incomplete:

| Status | Typical `problem` `title` | When |
|---|---|---|
| `400` | `"Malformed Request"` | Invalid JSON, wrong `Content-Type` |
| `401` | `"Unauthorized"` | Missing/invalid/expired credentials |
| `403` | `"Forbidden"` | Authenticated but not permitted for this resource |
| `404` | `"<Resource> Not Found"` | Resource doesn't exist |
| `409` | `"Conflict"` | Duplicate unique field, stale optimistic-lock version |
| `422` | `"Validation Failed"` | Field-level validation errors — use `errors[]` |
| `429` | `"Rate Limit Exceeded"` | See `auth-and-security.md` for headers |
| `500` | `"Internal Server Error"` | Never expose `detail` with stack traces or internals here |

`assets/problem-responses.json` has copy-pasteable bodies for each of these.
