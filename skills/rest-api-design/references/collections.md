# REST API Design — Collections: Pagination, Filtering, Bulk Ops

> Covers everything specific to endpoints that return more than one resource. For the shape of a
> single item, see `payloads-and-errors.md`.

## Table of Contents

1. [Cursor pagination (default)](#cursor-pagination-default)
2. [Offset pagination (escape hatch)](#offset-pagination-escape-hatch)
3. [Page size limits](#page-size-limits)
4. [Filtering](#filtering)
5. [Sorting](#sorting)
6. [Sparse fieldsets and includes](#sparse-fieldsets-and-includes)
7. [Total counts](#total-counts)
8. [Bulk create/update](#bulk-createupdate)
9. [Long-running operations](#long-running-operations)

---

## Cursor pagination (default)

Use an opaque cursor for any collection that can grow or mutate — it's stable under concurrent writes,
which offset pagination is not (see gotcha #5 in `SKILL.md`):

```http
GET /invoices?limit=25
→ 200 OK
{
  "data": [ ... 25 items ... ],
  "pagination": { "nextCursor": "eyJpZCI6Imludl8wMUg4In0", "hasMore": true }
}

GET /invoices?limit=25&cursor=eyJpZCI6Imludl8wMUg4In0
```

The cursor is opaque to the client — typically a base64-encoded pointer to the last-seen sort key
(e.g. `{id, createdAt}`), not something the client should construct or parse. Never expose it as a raw
offset or ID the client might infer meaning from.

## Offset pagination (escape hatch)

Legitimate when the collection is small, rarely mutated, and clients need to jump to an arbitrary
page (e.g. an admin table with a page-number UI):

```http
GET /countries?page=2&pageSize=50
→ 200 OK
{
  "data": [ ... ],
  "pagination": { "page": 2, "pageSize": 50, "totalPages": 5, "totalCount": 240 }
}
```

Document the trade-off if you choose this: pages shift under concurrent inserts/deletes.

## Page size limits

Every collection endpoint needs both a **default** and a **maximum** page size — never let a client
request an unbounded response:

```http
✅ correct
GET /invoices               → defaults to limit=25
GET /invoices?limit=500     → 422, limit exceeds max of 100

❌ wrong — no cap, a client (or bug) can request the entire table
GET /invoices?limit=1000000
```

## Filtering

Use query parameters named after the field, with simple, consistent operators for anything beyond
equality:

```http
GET /invoices?status=paid
GET /invoices?createdAt[gte]=2026-01-01&createdAt[lt]=2026-02-01
GET /invoices?amountCents[gt]=10000
```

Keep the operator grammar small and document it once (`[gte]`, `[lte]`, `[gt]`, `[lt]`, `[ne]`, bare
`=` for equality) rather than inventing per-field syntax. Reject unknown filter fields/operators with
`422` instead of silently ignoring them — a silently-ignored filter reads to the client as "matched
nothing" when actually the filter was never applied.

## Sorting

```http
GET /invoices?sort=createdAt        # ascending
GET /invoices?sort=-createdAt       # descending (leading -)
GET /invoices?sort=status,-createdAt  # multi-key
```

Document the whitelist of sortable fields — don't expose every internal column, and reject an
unsupported `sort` value with `422` rather than ignoring it.

## Sparse fieldsets and includes

Let clients opt into exactly the fields/relations they need, rather than always returning everything
or requiring a second round trip:

```http
GET /invoices?fields=id,status,amountCents
GET /invoices/{id}?include=customer,lineItems
```

Default (no `fields`/`include` given) to the full standard representation without relations embedded —
see `resource-modeling.md` → Relationships and embedding.

## Total counts

`totalCount` is often expensive (a full table scan/`COUNT(*)` on a filtered query) and clients rarely
need exact precision. Make it opt-in rather than computing it on every request:

```http
GET /invoices?limit=25&includeTotalCount=true
```

Cursor pagination's `hasMore` boolean is enough for "is there a next page?" UIs — reserve `totalCount`
for cases that genuinely need a number (e.g. "Showing 1-25 of 4,213").

## Bulk create/update

Model bulk operations as their own endpoint accepting an array, not as N calls to the single-item
endpoint — and return per-item results so a partial failure doesn't hide which items succeeded:

```http
POST /invoices/bulk
{ "items": [ { "customerId": "cus_1", ... }, { "customerId": "cus_2", ... } ] }

→ 207 Multi-Status
{
  "results": [
    { "status": 201, "data": { "id": "inv_1", ... } },
    { "status": 422, "error": { "title": "Validation Failed", "errors": [...] } }
  ]
}
```

`207 Multi-Status` signals "the request as a whole was processed, check each item." Document that bulk
endpoints are **not** atomic by default — if you need all-or-nothing semantics, say so explicitly and
make it a transaction server-side.

## Long-running operations

When an operation can't complete within a normal request timeout (report generation, bulk export,
async processing), don't block the request — accept it and let the client poll:

```http
POST /reports
→ 202 Accepted
  Location: /report-jobs/job_01H8X...
  { "id": "job_01H8X...", "status": "pending" }

GET /report-jobs/job_01H8X...
→ 200 OK
  { "id": "job_01H8X...", "status": "completed", "resultUrl": "/reports/rpt_01H9Y..." }
```

The job resource's `status` field (`pending`/`processing`/`completed`/`failed`) is the client's polling
signal — don't make the client guess completion from a timeout.
