# REST API Design — Resource Modeling

> Covers how to name and shape resources. For which HTTP method/status code to use once a resource is
> identified, see `http-semantics.md`.

## Table of Contents

1. [Nouns, not verbs](#nouns-not-verbs)
2. [Collections and items](#collections-and-items)
3. [Nesting sub-resources](#nesting-sub-resources)
4. [Identifiers](#identifiers)
5. [Relationships and embedding](#relationships-and-embedding)
6. [Modeling actions that aren't CRUD](#modeling-actions-that-arent-crud)
7. [Singleton resources](#singleton-resources)
8. [When REST is the wrong shape](#when-rest-is-the-wrong-shape)

---

## Nouns, not verbs

A REST path names a **resource** (a thing), and the HTTP method names the **operation** on it. The
method carries the verb — the path never should.

```http
✅ correct
GET    /invoices
POST   /invoices
GET    /invoices/{id}
DELETE /invoices/{id}

❌ wrong
GET    /getInvoices
POST   /createInvoice
POST   /invoices/delete
```

If you find yourself wanting a verb in the path, you're usually looking at one of:
- a **collection** operation (`POST /invoices` to create) — use the method instead
- a **non-CRUD action** on an existing resource — see [Modeling actions that aren't CRUD](#modeling-actions-that-arent-crud)
- an operation that doesn't fit REST at all — see [When REST is the wrong shape](#when-rest-is-the-wrong-shape)

## Collections and items

- **Collection**: `/invoices` — a set of resources. `GET` lists, `POST` creates.
- **Item**: `/invoices/{invoiceId}` — one resource. `GET` reads, `PUT`/`PATCH` updates, `DELETE` removes.

Use plural nouns for collections consistently (`/invoices`, not `/invoice`), even when an item route
hangs off it (`/invoices/{invoiceId}`). Don't mix singular and plural across an API.

## Nesting sub-resources

Nest a path only when the child **cannot exist without** the parent and is always scoped to it:

```http
✅ correct — line items only make sense inside an invoice
GET  /invoices/{invoiceId}/line-items
POST /invoices/{invoiceId}/line-items

❌ wrong — a customer exists independently; don't force every lookup through invoices
GET /invoices/{invoiceId}/customer/orders
```

**Cap nesting at two levels deep.** Beyond `/parents/{id}/children/{id}`, flatten by giving the
grandchild its own top-level collection and filtering:

```http
✅ correct
GET /line-item-adjustments?lineItemId={id}

❌ wrong — too deep, brittle to reorganize
GET /invoices/{invoiceId}/line-items/{lineItemId}/adjustments/{adjustmentId}/notes
```

## Identifiers

- Prefer **opaque, non-guessable IDs** (UUIDv4/v7 or ULID) over sequential integers for anything
  exposed to external clients — see `auth-and-security.md` for why (enumeration).
- IDs are always strings in JSON, even when internally numeric, so clients never lose precision or
  accidentally do integer math on them: `"id": "018f4d2e-..."`, not `"id": 018f4d2e`.
- A resource's ID never changes after creation. If you need a human-friendly alias (a slug, an
  order number), expose it as a *separate* field, not as the identity used in the URL.
- Use the same ID field name everywhere the resource appears (`invoiceId` in a nested body, `id` in
  its own representation, `{invoiceId}` in the path) — pick one pattern and hold it.

## Relationships and embedding

Represent a relationship to another resource as its ID (or a short reference object), and let the
client fetch the full related resource separately or via an explicit include:

```json
✅ correct — reference by ID, optionally expandable
{
  "id": "inv_01H...",
  "customerId": "cus_01H...",
  "customer": null
}
```

```http
GET /invoices/{id}?include=customer
```

```json
❌ wrong — always embedding the full related object bloats every response and hides N+1s
{
  "id": "inv_01H...",
  "customer": { "id": "cus_01H...", "name": "...", "address": {...}, "orders": [ ... ] }
}
```

Sparse, opt-in embedding (`?include=customer,lineItems`) beats always-embed or never-embed — see
`collections.md` for sparse fieldsets and includes on collection endpoints too.

## Modeling actions that aren't CRUD

Not everything is create/read/update/delete. For a state transition or side-effecting action on an
existing resource, model it as a sub-resource **action verb** the resource collection doesn't
otherwise use, invoked with `POST`:

```http
✅ correct
POST /orders/{id}/cancel
POST /invoices/{id}/send
POST /subscriptions/{id}/pause

❌ wrong — bolting a status enum onto PATCH to trigger side effects
PATCH /orders/{id}   { "status": "cancelled" }   # looks like a plain field update but
                                                   # actually triggers refunds, emails, etc.
```

Reserve `PATCH` for genuine partial *data* updates. If setting a field triggers a side effect
(refund, notification, external call), model it as its own action endpoint instead — it makes the
side effect discoverable, documentable, and separately authorizable.

## Singleton resources

Some resources have exactly one instance per parent — no collection, no ID needed:

```http
GET /users/{id}/settings
PUT /users/{id}/settings
```

Don't invent a collection or fake ID for something that's inherently 1:1 (`/users/{id}/settings/{settingsId}`).

## When REST is the wrong shape

Not every operation belongs in a resource-oriented API. Reach for something else when:

- **The operation is a pure computation with no persistent resource** (e.g. "validate this address,"
  "calculate shipping cost") — a single action-style `POST` endpoint is fine; don't force a fake resource.
- **Clients need to combine many resources in one round trip with variable shape** — consider GraphQL
  instead of proliferating `?include=` combinations.
- **The interaction is fundamentally a stream of events other systems react to**, not a request/response
  a client polls for — consider webhooks or an event/message system instead of a REST resource.
- **Two services need a tightly-coupled, low-latency call with strict typing** — consider gRPC.

Naming the mismatch explicitly to the user beats forcing every operation into `GET`/`POST`/`PATCH`
and ending up with an API that's "RESTish" in shape but not actually resource-oriented in spirit.
