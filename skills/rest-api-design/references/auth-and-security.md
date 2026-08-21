# REST API Design — Auth & Security

> Covers authentication, authorization, rate limiting, and the security pitfalls specific to REST
> APIs. For general web app security beyond the API surface, use a dedicated security review.

## Table of Contents

1. [Choosing an auth mechanism](#choosing-an-auth-mechanism)
2. [JWT vs opaque tokens](#jwt-vs-opaque-tokens)
3. [Authorization: scopes, roles, and object-level checks](#authorization-scopes-roles-and-object-level-checks)
4. [CORS](#cors)
5. [Rate limiting](#rate-limiting)
6. [Input validation and payload limits](#input-validation-and-payload-limits)
7. [Common pitfalls](#common-pitfalls)

---

## Choosing an auth mechanism

| Mechanism | Use for |
|---|---|
| OAuth 2.1 / OIDC | User-facing APIs where a third party (or your own SPA/mobile app) acts on behalf of a user |
| Client credentials grant | Service-to-service, no end user involved |
| API keys | Simple server-to-server integrations, internal tooling; weaker than OAuth — no scoping/expiry by default |
| Session cookies | Same-origin web app calling its own backend — pair with CSRF protection |

Every request to a non-public endpoint must carry credentials in the `Authorization` header (`Bearer
<token>`) or an equivalent — never in the URL/query string, where it leaks into logs, browser history,
and `Referer` headers:

```http
✅ correct
GET /invoices
Authorization: Bearer eyJhbGciOi...

❌ wrong — token in the URL
GET /invoices?api_key=sk_live_abc123
```

## JWT vs opaque tokens

- **JWT** — self-contained, verifiable without a database round trip, good for short-lived access
  tokens across services. Cannot be revoked before expiry without extra infrastructure (a denylist) —
  keep the TTL short (minutes, not days) and pair with a refresh token.
- **Opaque token** — a random string looked up server-side. Trivially revocable, but every request
  costs a lookup (or requires a cache). Good default for session-like tokens with a longer lifetime.

Don't put sensitive data (PII, permissions that change frequently) in a JWT payload — it's
base64-encoded, not encrypted, and readable by anyone holding the token.

## Authorization: scopes, roles, and object-level checks

Authorization is two separate questions — answer both, not just the first:

1. **Is this caller allowed to perform this kind of action at all?** (scopes/roles: `invoices:write`,
   `admin`)
2. **Is this caller allowed to perform it on *this specific* resource?** (object-level: does this
   invoice belong to this caller's account?)

```http
❌ wrong — checks the scope, not the object
GET /invoices/{id}
# server checks: does the caller have `invoices:read`? → yes → returns ANY invoice by ID,
# including ones belonging to a different account (IDOR — see Common pitfalls)

✅ correct
GET /invoices/{id}
# server checks: does the caller have `invoices:read`? AND
# does invoice {id} belong to the caller's account/org? → both must hold
```

Skipping the object-level check is the single most common REST API authorization bug (IDOR — see
below). Every handler for `/{resource}/{id}` needs an explicit ownership/tenancy check, not just an
authentication check.

## CORS

For browser-based clients on a different origin, respond to preflight `OPTIONS` requests with an
explicit allowlist — never `Access-Control-Allow-Origin: *` on any endpoint that requires
authentication (credentials and wildcard origins are mutually exclusive per spec, and combining them
incorrectly is a common misconfiguration):

```http
✅ correct
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true

❌ wrong — wildcard origin with a credentialed endpoint
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

## Rate limiting

Return `429 Too Many Requests` with headers telling the client how to behave, rather than just
dropping the connection or returning a generic `500`:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 30
Content-Type: application/problem+json

{ "type": "about:blank", "title": "Rate Limit Exceeded", "status": 429 }
```

Apply limits per API key/user, not just per IP (shared NATs/corporate proxies punish every user behind
them equally under IP-only limits). Document the limit in the OpenAPI spec or developer docs so
clients can implement backoff proactively instead of discovering it in production.

## Input validation and payload limits

- Validate every field server-side regardless of client-side validation — the client is not trusted.
- Reject unknown fields in strict-schema APIs (or explicitly document that extras are ignored) —
  silently accepting unknown fields both hides typos and enables mass-assignment (see below).
- Cap request body size (most frameworks default this; verify it's not disabled) to prevent a single
  oversized payload from exhausting memory.
- Cap array lengths in request bodies (e.g. bulk endpoints) independently of the byte-size cap.

## Common pitfalls

1. **IDOR (Insecure Direct Object Reference)** — authenticating the caller but not checking that the
   specific object ID in the path/body belongs to them. See the object-level check above; this is the
   #1 API authorization bug in practice.

2. **Mass assignment** — binding the entire request body directly onto a model/entity without an
   explicit allowlist of writable fields, letting a client set fields it shouldn't (`role: "admin"`,
   `accountBalance: 999999`) just by including them in the JSON.
   ```http
   ❌ wrong — client sets a field the API never intended to expose
   PATCH /users/{id}
   { "displayName": "New Name", "role": "admin" }
   ```
   Always validate against an explicit writable-fields allowlist per endpoint, never a blanket
   deserialize-and-save.

3. **Enumeration via sequential IDs or verbose errors** — see `SKILL.md` gotcha #10; pair non-guessable
   IDs with authorization checks that return the same response (typically `404`) whether the resource
   doesn't exist or the caller can't see it, so the response itself doesn't confirm existence.

4. **Verbose error messages** — stack traces, ORM/SQL error text, or internal file paths in a `500`
   body hand an attacker a map of your internals. Log the detail server-side; return a generic
   `problem` body to the client (see `payloads-and-errors.md`).

5. **Secrets or tokens in URLs** — query-string credentials end up in access logs, browser history, and
   `Referer` headers sent to third parties. Always use `Authorization` headers instead.

6. **Missing TLS/HSTS** — every API endpoint must be HTTPS-only; redirect or reject plain HTTP, and set
   `Strict-Transport-Security` so clients don't silently downgrade after the first request.
