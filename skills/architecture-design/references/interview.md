# The Interview

This file is the single source of truth for what the skill asks. `SKILL.md` only summarizes it.

## Rules that apply to every phase

- **One phase per turn.** Ask the phase's questions, wait for the answer, then move on. Never batch two phases.
- **At most four questions per turn.** Pick the ones whose answers change the design; drop the rest or fold them into follow-ups.
- **Every question ships a default.** Phrase it so "yes" or "all defaults" is a complete answer: `How many users in year one? (If unsure: < 1,000, and I'll design for 10×.)`
- **Skip what's already answered.** Anything the request, the repo, or a previous answer settled is confirmed in a single line, not asked again.
- **Follow-ups are conditional.** Only ask them when the trigger fires. Unanswered follow-ups become open questions in the doc, not assumptions.
- **Write the answer down as you go.** Keep a running fact list; the synthesis phase consumes it.

## Phase 1 — Problem & goals

**Goal:** know what the system is for and what "v1 succeeded" means, in the user's words.

Core questions:
1. In one or two sentences, what does this system do and for whom?
2. What does success look like three months after launch? (Default: "N paying customers / active users using the core flow weekly".)
3. What is explicitly **not** a goal for v1? (Default: none stated — I'll propose non-goals after scope.)
4. Is this replacing an existing system, or new? (Default: new.)

Follow-ups:
- Replacing an existing system → what must stay compatible (data, URLs, integrations, users' habits)? Is there a migration window or a hard cutover?
- The user describes a product, not a system → ask what is *software* here vs. process; keep the interview on the software.

## Phase 2 — Users & scale

**Goal:** who the users are and the realistic load — enough to size, not to over-engineer.

Core questions:
1. Who are the user types (roles)? Are any of them external organizations (B2B) as opposed to individuals (B2C)? (Default: one internal admin role + one end-user role.)
2. How many users in the first year, and how many concurrent at peak? (Default: < 1,000 users, < 50 concurrent; design for 10×.)
3. Where are they — one country/region, or global? (Default: one region.)
4. Are there traffic spikes you already know about (launches, month-end, events)? (Default: none.)

Follow-ups:
- B2B / multiple organizations → is data strictly isolated per organization (multi-tenant)? Do organizations ever share data? Do they need their own subdomains, branding, SSO?
- Global users → is latency across regions a product concern, or is one region acceptable for v1?
- Load > 10k concurrent or > 1M users in year one → treat scale as a driver and ask what the read/write mix looks like.

Skip when: the repo or request already fixes tenancy (e.g. an existing `organizations` table with `tenant_id` everywhere).

## Phase 3 — Functional scope

**Goal:** the v1 feature list and what is deliberately deferred.

Core questions:
1. List the core flows v1 must support (the 3–7 things a user does). (No default — this one the user must answer.)
2. Which of those is the "money" flow — the one the business fails without?
3. What is deferred to v2+? (Default: I'll propose a cut based on the list.)
4. Any features that are unusual for this kind of product — real-time collaboration, offline use, heavy file/media processing, ML/AI, payments, marketplaces with two sides? (Default: none.)

Follow-ups:
- Real-time → what latency counts as real-time here (sub-second vs. "within a minute")? Is it collaboration (conflicts) or just live updates?
- Offline → which flows must work offline, and what happens on conflict?
- Payments → who is the processor, are you the merchant of record, do you need subscriptions, invoices, or payouts?
- File/media → expected sizes, formats, and whether processing must be synchronous.
- AI/LLM features → is it a core flow or an enhancement? Latency and cost tolerance per call? Any data that must not leave your infrastructure?
- Marketplace / two-sided → matching, search, and trust/ratings usually dominate the design; ask which side v1 optimizes for.

## Phase 4 — Constraints

**Goal:** what the design must live with, regardless of what is technically ideal.

Core questions:
1. Team: how many engineers, and what stack do they know best? (Default: 2–3 engineers; stack = whatever the repo already uses, else the team's strongest.)
2. Deadline for v1 and what's driving it (customer commitment, funding, event)? (Default: 3 months, soft.)
3. Budget for infrastructure per month, roughly? (Default: < 500 USD/month.)
4. Compliance or data-residency requirements — GDPR, HIPAA, PCI, SOC 2, local laws, data must stay in country X? (Default: none beyond standard privacy hygiene.)

Follow-ups:
- Existing systems it must integrate with or replace → list them, their interfaces (API, DB, files, email), and who owns them.
- Compliance named → which data is in scope, whether audits are expected in the first year, whether there's an existing policy to align to.
- Team knows stack A but the request says stack B → ask which matters more, learning curve or the reason for B; record the answer as a decision.
- Mandated vendor, cloud, or platform ("must run on our Azure", "must use Salesforce as CRM") → record as a hard constraint.

Skip when: the repo fixes the stack; the request states the team or deadline.

## Phase 5 — Quality attributes

**Goal:** a ranked list of the non-functional requirements — not everything "high".

Core questions:
1. If the system is down for an hour on a weekday, what happens? (Default: annoyance, no revenue lost → 99.5% is fine, no multi-region.)
2. What's the slowest acceptable response for the money flow? (Default: < 1 s for interactive pages, < 5 s for reports.)
3. What data would be a disaster to lose or leak? (Default: user PII and anything financial.)
4. Rank these for v1: correctness/consistency, availability, latency, cost, time to ship. (Default: time to ship > correctness > cost > availability > latency.)

Follow-ups:
- Availability ranked first → confirm the RTO/RPO in numbers; ask whether the business will actually pay for multi-AZ/multi-region.
- Consistency critical (money, inventory, bookings) → identify the invariants that must never break (no double-booking, balances always sum) — they drive transaction boundaries.
- Sensitive data → encryption at rest requirements, who may see it, audit log needs, retention limits.
- "Everything is high" → push back once: ask which one they'd give up first.

## Phase 6 — Data & integrations

**Goal:** the shape and volume of data and every external system.

Core questions:
1. What are the 5–10 core entities and how do they relate? (Default: I'll propose from the flows.)
2. Expected data volume in year one — rows in the biggest table, total storage, files? (Default: < 10 M rows, < 100 GB, files < 1 TB.)
3. Which third-party services are already decided or likely (auth, email, payments, SMS, maps, analytics, AI providers)? (Default: none decided.)
4. Does anyone else need to call *your* system (public API, webhooks, partners)? (Default: no public API in v1.)

Follow-ups:
- Large volumes or time-series/event data → ask about retention, whether old data must stay queryable, and whether analytics is a v1 need.
- Search as a core flow → full-text over what, how fresh must results be, do you need faceting/relevance tuning?
- Public API in v1 → who consumes it, versioning expectations, auth model for third parties.
- Data imports from the existing system → format, one-time vs. ongoing, and data quality known issues.
- Reporting/analytics → who reads them, how fresh, do they hit the production DB.

## Phase 7 — Delivery & operations

**Goal:** how it gets built, shipped, and kept alive.

Core questions:
1. Where will it run — a mandated cloud, a PaaS (Fly, Render, Heroku, Railway), a VPS, on-prem? (Default: whatever the repo deploys to; else a PaaS or a Kamal-style VPS deploy.)
2. Environments needed beyond production — staging, per-PR previews? (Default: production + staging.)
3. Who is on call, and what do they need to see when something breaks? (Default: the developers; error tracking + logs + uptime check.)
4. Any existing CI/CD, container, or IaC conventions to reuse? (Default: reuse what the repo has; else GitHub Actions.)

Follow-ups:
- On-prem or customer-hosted → packaging, upgrade path, and support burden become drivers.
- No one is on call → design for self-healing basics (managed DB with backups, restart policies) and say so in the doc.
- Mobile clients → app-store release cadence affects API compatibility; ask how many app versions must stay supported.

## Enough to propose — the gate

Move to synthesis only when all of these are true. Anything missing is either asked next turn or written into "Risks & open questions".

- [ ] One-sentence purpose and a success criterion (phase 1)
- [ ] User types, tenancy model, and an order-of-magnitude load (phase 2)
- [ ] The core flows and the money flow; a v1 / later cut (phase 3)
- [ ] Team size and stack, deadline, budget, compliance constraints (phase 4)
- [ ] A ranked quality-attribute list with at least the top two in numbers (phase 5)
- [ ] Core entities, approximate volume, decided third-party services, whether there's a public API (phase 6)
- [ ] Hosting target, environments, minimum observability (phase 7)

If the user says "just propose something" before the gate is met, propose — but fill each unmet item with the phase default, label every one of them as an assumption in the doc, and list them under open questions.
