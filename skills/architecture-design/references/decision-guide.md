# Decision Guide

Heuristics for turning drivers into decisions. The rule throughout: **choose the simplest option that satisfies the drivers, and write down the driver that justified anything more complex.**

## Structure — monolith, modular monolith, services

| Choose | When |
|---|---|
| **Modular monolith** (default) | Almost every v1: one deployable, one database, code organized by domain module with explicit boundaries |
| Monolith + separate worker | Background work has very different scaling or runtime needs (video encoding, ML inference) — still one codebase |
| Services from day one | A driver forces it: separate teams with separate release cadence, a component with a hard runtime/language constraint, or a regulatory boundary that must be a process boundary |
| Serverless functions | Bursty, event-shaped, low-state workloads; a team that already operates that way; strong cost sensitivity at low volume |

```text
❌ wrong — "Microservices for scalability" on a 3-person team with < 1,000 users.
   Cost: distributed transactions, N deploy pipelines, network failure modes, no driver satisfied.

✅ correct — Modular monolith with domain modules (billing/, scheduling/, accounts/),
   each owning its tables, communicating through in-process interfaces.
   Extraction to a service later is a refactor, not a rewrite.
```

Module boundaries matter more than deployment boundaries in v1: name the modules in the doc and state which module owns which entities.

## Language & framework

- The team's strongest stack wins unless a driver forbids it. Learning a new framework and shipping a v1 in three months are competing goals.
- An existing repo has already decided. Confirm, don't reopen.
- Prefer a batteries-included framework (Rails, Django, Laravel, NestJS, Phoenix) for CRUD-heavy products: auth, ORM, migrations, jobs, and mailers come for free.
- Record the choice with the alternative the team considered — even if the answer was obvious.

## Storage

| Need | Default |
|---|---|
| Primary data | One relational database (PostgreSQL unless the platform dictates otherwise) |
| Files / media | Object storage (S3-compatible) with signed URLs — never the DB, never local disk |
| Cache / sessions / rate limits | Redis, only once a measured need appears; many v1s don't need it |
| Full-text search | Postgres full-text first; a dedicated engine (Meilisearch, Elasticsearch/OpenSearch) only when relevance tuning or facets are a core flow |
| Time-series / events / analytics | Postgres tables with partitioning first; a warehouse or column store when analytics is a product feature or volume passes what one DB handles |
| Document / key-value | Usually a `jsonb` column; a separate document DB only for a driver the relational model can't meet |

A second database in v1 needs a driver named in the decision log. "We might need it later" is not one.

## Background work & async

- Anything slower than ~1 s or talking to a third party goes in a **job queue** (Sidekiq/GoodJob, BullMQ, Celery, Oban…). Jobs must be idempotent and retried with backoff.
- Scheduled work → the queue's scheduler, not cron on a box.
- Event-driven architecture, a message broker (Kafka, RabbitMQ), or event sourcing only when a driver needs it: multiple consumers of the same events across deployables, audit/replay as a product requirement, or integration fan-out at scale.
- Real-time to the browser → the framework's WebSocket layer (Action Cable, Phoenix Channels, Socket.IO) or server-sent events; a hosted pub/sub (Ably, Pusher) if the team doesn't want to operate sockets.

## API style

- Server-rendered HTML (with Hotwire/HTMX/Livewire) or a single SPA — pick by team skill and product need, not fashion. Server-rendered ships faster for CRUD; an SPA when the UI is highly interactive or mobile shares the API.
- Internal API for your own clients: REST/JSON is the default; GraphQL when many client shapes are a real need; tRPC-style when the whole stack is TypeScript.
- Public API in v1 → REST, versioned from day one, with API keys or OAuth client credentials, rate limits, and a published OpenAPI spec. (The `rest-api-design` skill covers the details.)

## Auth, authorization & tenancy

- **Authentication:** the framework's own (Devise, Django auth, NextAuth/Auth.js) or a hosted provider (Auth0, Clerk, Cognito, Firebase Auth) if the team wants to avoid owning passwords and MFA. B2B with enterprise customers → plan for SSO (SAML/OIDC) even if v1 doesn't ship it.
- **Authorization:** role-based per user type in v1; a policy layer (Pundit, CASL, Cerbos) once rules cross entities. Write down who can see what.
- **Multi-tenancy:** shared database with a `tenant_id` on every tenant-owned table, enforced at the query layer (default scopes / row-level security), is the v1 answer. Schema-per-tenant or database-per-tenant only for a compliance or isolation driver, and note the operational cost.

```text
❌ wrong — Tenant isolation left to "we'll remember to filter by organization in every query".
✅ correct — Every tenant-owned table carries tenant_id; the ORM applies a tenant scope from the
   request context; Postgres row-level security as a second line for sensitive tables.
```

## Hosting & deployment

| Situation | Default |
|---|---|
| Mandated cloud/platform | Use it; note the constraint |
| Small team, no ops appetite | A PaaS (Fly.io, Render, Railway, Heroku) with a managed database |
| Cost-sensitive, some ops comfort | Containers on a VPS with Kamal or similar, managed database from a provider |
| Existing Kubernetes with a platform team | Use it — but v1 doesn't *introduce* Kubernetes |

Non-negotiables in v1 regardless of host: a managed or at least automatically backed-up database with tested restore, environment-based config with no secrets in the repo (placeholders like `your-token` in examples), a `staging` environment, one-command deploy from CI, and a rollback path.

## Observability minimum

Error tracking (Sentry/Bugsnag/Honeybadger), structured logs shipped somewhere searchable, an uptime check on the money flow, and basic metrics from the host. Tracing and dashboards when there's someone to read them. Name who gets paged, even if the answer is "nobody, we look in the morning".

## Security baseline

TLS everywhere; secrets in the host's secret store; dependency audit in CI; framework CSRF/XSS defaults left on; least-privilege DB and cloud credentials; PII inventory in the doc with retention rules; backups encrypted. Compliance drivers (GDPR, HIPAA, PCI) add: data-subject deletion path, audit log, encryption at rest for named fields, and — for PCI — never touching card data (hosted payment fields).

## Driver-specific patterns

- **Payments:** hosted checkout/fields from the processor; webhooks with signature verification and idempotency keys; a `payments` module that owns all money tables; reconciliation job.
- **Files & media:** direct-to-object-storage uploads with signed URLs; processing in jobs; store derived assets with a content hash.
- **AI/LLM features:** a thin provider abstraction; every call in a job or streamed with a timeout; log prompts/costs; a kill switch; never send data the compliance driver forbids.
- **Search-centric products:** decide the engine in v1 — it shapes the data model.
- **Offline / mobile-first:** an API designed for sync (versioned records, tombstones) from the start.
- **Reporting:** read replicas or materialized views before a warehouse.

## What NOT to build in v1

Unless a named driver demands it: microservices, Kubernetes, a message broker, event sourcing/CQRS, a second database, multi-region, a custom auth system, a design-system package, a GraphQL gateway, a plugin architecture, feature flags service (a config table is fine), a data warehouse, an admin framework of your own (use the framework's or a hosted one).

Put each of these that the user asked for and you talked them out of in the **Evolution path** section, with the trigger that would justify it later.
