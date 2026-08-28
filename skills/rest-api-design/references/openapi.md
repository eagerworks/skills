# REST API Design — OpenAPI 3.1

> Covers writing and maintaining an OpenAPI spec. For the design decisions the spec should encode
> (methods, status codes, error shapes), see `http-semantics.md` and `payloads-and-errors.md`.

## Table of Contents

1. [Contract-first workflow](#contract-first-workflow)
2. [Document structure](#document-structure)
3. [Reusing schemas with $ref](#reusing-schemas-with-ref)
4. [Documenting every reachable response](#documenting-every-reachable-response)
5. [Security schemes](#security-schemes)
6. [Examples](#examples)
7. [Webhooks](#webhooks)
8. [Linting](#linting)
9. [Codegen and mock servers](#codegen-and-mock-servers)
10. [Keeping the spec in sync](#keeping-the-spec-in-sync)

---

## Contract-first workflow

Prefer writing (or updating) the OpenAPI spec **before** implementation for any new endpoint or
breaking change: it forces the response/error shape and edge cases to be decided up front, gives
frontend/consumer teams something to build against immediately (even mocked), and becomes the diff you
review for breaking changes. Retrofitting a spec from code after the fact tends to just document
whatever the implementation happened to do, including its bugs.

## Document structure

```yaml
openapi: 3.1.0
info:
  title: Example API
  version: 1.0.0
servers:
  - url: https://api.example.com/v1
paths:
  /invoices:
    get: { ... }
    post: { ... }
  /invoices/{invoiceId}:
    get: { ... }
components:
  schemas: { ... }
  responses: { ... }
  parameters: { ... }
  securitySchemes: { ... }
```

Put every reusable piece — schemas, common responses, common parameters — under `components/` and
`$ref` it from `paths/`. See `assets/openapi.starter.yaml` for a full worked example.

## Reusing schemas with $ref

Never redefine the same object shape inline in multiple places — it drifts:

```yaml
✅ correct
components:
  schemas:
    Invoice:
      type: object
      properties:
        id: { type: string }
        status: { type: string, enum: [draft, sent, paid] }
paths:
  /invoices/{invoiceId}:
    get:
      responses:
        '200':
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Invoice' }

❌ wrong — the same shape typed out again in every endpoint that returns an invoice
```

## Documenting every reachable response

List every status code the endpoint can actually return, not just the happy path — this is where
`references/http-semantics.md` and `payloads-and-errors.md` become the spec:

```yaml
paths:
  /invoices/{invoiceId}:
    get:
      responses:
        '200':
          description: Invoice found
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Invoice' }
        '404':
          $ref: '#/components/responses/NotFound'
        '401':
          $ref: '#/components/responses/Unauthorized'
```

Define `NotFound`, `Unauthorized`, `ValidationError`, etc. once under `components/responses` (each
wrapping the `Problem` schema from `payloads-and-errors.md`) and reuse them across every endpoint.

## Security schemes

```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
security:
  - bearerAuth: []
```

Set `security` globally at the document root, and override per-operation (`security: []` for a public
endpoint) only where it genuinely differs — don't repeat the same scheme on every single path.

## Examples

Attach a realistic `example` (or `examples` for multiple cases) to every schema and response — it
doubles as documentation and as fixture data for mock servers and contract tests:

```yaml
Invoice:
  type: object
  properties:
    id: { type: string }
    status: { type: string, enum: [draft, sent, paid] }
  example:
    id: inv_01H8X3Z9QK7N2M4P
    status: paid
```

## Webhooks

Document outbound events the same way as inbound endpoints, using OpenAPI 3.1's `webhooks` keyword —
consumers need to know the event payload shape just as much as request/response shapes:

```yaml
webhooks:
  invoicePaid:
    post:
      requestBody:
        content:
          application/json:
            schema: { $ref: '#/components/schemas/Invoice' }
```

## Linting

Lint the spec in CI with [Spectral](https://github.com/stoplightio/spectral) (or an equivalent) so
drift and inconsistency are caught automatically rather than in review:

```bash
npx @stoplight/spectral-cli lint openapi.yaml
```

A minimal ruleset to enforce: every operation has an `operationId`, every response has a `description`,
no duplicate `operationId`s, all `$ref`s resolve.

## Codegen and mock servers

Once the spec is accurate, generate value from it instead of hand-writing what it already encodes:
- **Client SDKs** — `openapi-generator` or similar, so consumers don't hand-write HTTP calls.
- **Mock servers** — Prism or similar, so frontend work can start against the contract before the
  backend implementation exists.
- **Server-side validation** — some frameworks can validate incoming requests against the spec
  directly, catching contract drift at the boundary.

## Keeping the spec in sync

The spec is only useful if it matches reality. Options, roughly in order of reliability:
1. **Contract tests** that assert real responses conform to the spec's schemas (strongest).
2. **Spec-driven server validation** (framework validates requests/responses against the spec at
   runtime in non-prod).
3. **Manual discipline** — spec updated in the same PR as the endpoint change (weakest; drifts over
   time without one of the above).

When reviewing a PR that changes an endpoint, treat an out-of-date spec as part of the diff that's
missing — see `review-checklist.md`.
