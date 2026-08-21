# REST API Design — Versioning & Evolution

> Covers how to change an API that already has clients without breaking them.

## Table of Contents

1. [URL versioning (default)](#url-versioning-default)
2. [Alternative schemes: header and media-type versioning](#alternative-schemes-header-and-media-type-versioning)
3. [Breaking vs additive changes](#breaking-vs-additive-changes)
4. [The tolerant reader rule](#the-tolerant-reader-rule)
5. [Deprecation policy](#deprecation-policy)
6. [API lifecycle stages](#api-lifecycle-stages)

---

## URL versioning (default)

Put the major version in the path, and bump it only for breaking changes:

```http
GET /v1/invoices
GET /v2/invoices
```

Why it's the default: it's visible in logs, curl commands, and browser bars without inspecting
headers; it's trivial to route at the load balancer/gateway level; and it makes "which version am I
calling" unambiguous to every caller, including humans debugging by hand.

Deviate to an unversioned API only for a genuinely internal API with a single trusted client deployed
in lockstep with the server (see house style table in `SKILL.md`) — and even then, plan to add
versioning before a second client shows up.

## Alternative schemes: header and media-type versioning

Legitimate alternatives when the codebase already uses them — don't introduce a second scheme
alongside an existing one:

```http
✅ header versioning
GET /invoices
Api-Version: 2026-08-12

✅ media-type versioning
GET /invoices
Accept: application/vnd.example.v2+json
```

Header/media-type versioning keeps URLs stable across versions (useful for caching by URL) at the
cost of being invisible to casual inspection — pick it deliberately, not by accident.

## Breaking vs additive changes

| Change | Breaking? |
|---|---|
| Adding a new optional field to a response | No |
| Adding a new endpoint | No |
| Adding a new optional query parameter | No |
| Adding a new enum value to an existing field | **Yes**, for clients with a closed switch/enum |
| Removing or renaming a field | Yes |
| Changing a field's type (string → number) | Yes |
| Making an optional request field required | Yes |
| Changing a status code an endpoint returns | Yes |
| Changing the meaning of an existing field | Yes |
| Tightening validation on an existing field | Yes |

Adding an enum value is the subtle one: it's additive in principle, but any client with a `switch`
that doesn't have a `default` case will break. State this explicitly to API consumers in docs (see
[Tolerant reader rule](#the-tolerant-reader-rule)) rather than assuming "additive" always means "safe."

**When in doubt, treat it as breaking.** A false-positive version bump costs a changelog entry; a
false negative silently breaks every client that made a reasonable assumption about the old contract.

## The tolerant reader rule

Design clients (and instruct API consumers) to ignore fields they don't recognize and tolerate new
enum values gracefully (fall through to a default/unknown case) — this is what makes *additive*
changes actually safe in practice, not just in theory. Document this expectation in the API's public
docs so client authors build to it from day one.

## Deprecation policy

Announce, don't just remove:

```http
GET /v1/invoices/{id}
→ 200 OK
  Deprecation: true
  Sunset: Sat, 01 Aug 2026 00:00:00 GMT
  Link: </v2/invoices/{id}>; rel="successor-version"
```

- `Deprecation` header (or `true`) marks the endpoint/field as deprecated starting now.
- `Sunset` (RFC 8594) gives the date it stops working — give clients real time (weeks to months
  depending on your ecosystem, not days).
- `Link` with `rel="successor-version"` points at the replacement.
- Mirror this in a human-readable changelog, and — where the volume justifies it — proactively notify
  known API consumers (email, dashboard banner) rather than relying on them to read response headers.

For a deprecated *field* rather than a whole endpoint, note it in the OpenAPI spec
(`deprecated: true` on the schema property) and in the changelog; headers apply at the endpoint level.

## API lifecycle stages

1. **Design** — spec written and reviewed (see `openapi.md`, `review-checklist.md`) before
   implementation starts.
2. **Preview/beta** — available but explicitly unstable; state this in docs so consumers don't build
   production dependencies on it yet.
3. **Stable (`v1`, `v2`, ...)** — the breaking-change rules above apply; changes are additive-only
   within a major version.
4. **Deprecated** — still works, `Deprecation`/`Sunset` headers present, migration path documented.
5. **Sunset** — returns `410 Gone` (not silently `404`) with a `problem` body pointing at the
   successor, for a grace period, before the route is removed entirely.
