# Audit SEO — Workflow

> The six phases end to end, what's allowed to run and fetch, how to bound a crawl, and how
> to write the dated report file. This is the skill's procedural spine — `SKILL.md` only
> summarizes it.

## Table of Contents

1. [Phase 0 — Config](#phase-0--config)
2. [Phase 1 — Resolve the mode](#phase-1--resolve-the-mode)
3. [Phase 2 — Sample pages](#phase-2--sample-pages)
4. [Phase 3 — Gather evidence](#phase-3--gather-evidence)
5. [Phase 4 — Grade](#phase-4--grade)
6. [Phase 5 — Work Plan](#phase-5--work-plan)
7. [Phase 6 — Write the report](#phase-6--write-the-report)
8. [Re-running after fixes](#re-running-after-fixes)
9. [Allowed vs. forbidden actions](#allowed-vs-forbidden-actions)

---

## Phase 0 — Config

Read `.eagerworks/audit-seo.json` if present (`references/config.md`). It can move the
report directory, name explicit sample URLs, bound the crawl, disable a dimension, and
pre-declare facts (like Search Console verification) that would otherwise be ⚪. Every
effect it has on the report — a disabled dimension, a `known.*` fact taken as evidence —
gets disclosed in the footer, never applied silently.

## Phase 1 — Resolve the mode

Look at what the user actually gave you:

| Given | Mode | What you can grade |
|---|---|---|
| A URL (production or a reachable staging/preview) | **Live** | Everything dimension 1–8 can observe from HTTP responses and a rendered browser |
| A repo, no reachable URL | **Code** | Metadata source, routing, sitemap/robots generators — mark anything that needs a live response ⚪ |
| Both | **Combined** | Live evidence, traced back to the source line that produces it when a finding needs a fix |

If the user names a URL but you're sitting in a repo that plainly builds it, that's
**Combined** — use the code to explain *why* a live finding exists, not to replace fetching
it. Don't assume Combined just because a repo is open; confirm the URL actually serves that
repo's output before treating them as the same site (gotcha 4 — staging/preview vs.
production).

In Code mode, detect the framework from `references/framework-detection.md` before grading
anything — where metadata lives differs enough between a Next.js app router and a Rails
`content_for` partial that guessing produces wrong findings.

## Phase 2 — Sample pages

Never crawl the whole site (gotcha 5). Build a bounded sample:

1. The homepage.
2. One page per distinct template/route type you can identify (a blog post, a product
   page, a category/listing page, a landing page) — from `config.site.sampleUrls` if given,
   otherwise from the sitemap or the repo's routing.
3. Cap at `crawl.maxPages` (default 20). If the site has more distinct templates than the
   cap allows, sample the highest-traffic-looking ones (homepage, top nav destinations) and
   say in the report which templates were skipped.

Fetch serially with a delay (`crawl.requestDelayMs`, default 1000ms) and respect
`robots.txt` — this is never overridden by config (`references/config.md` → Rules).

## Phase 3 — Gather evidence

For each sampled URL, gather what `references/rubric.md` needs: raw HTML, response headers,
and — for dimension 6 specifically — the rendered DOM. `references/live-site-checks.md`
has the exact fetch recipes, what claude-in-chrome is for versus a plain HTTP fetch, and
the honest list of what a fetch-based audit cannot measure (real Core Web Vitals, search
volume, rankings).

## Phase 4 — Grade

Grade every check in `references/rubric.md` against the evidence gathered. Roll each
dimension up to its worst check. A grade needs the evidence that satisfies it cited
inline — a URL and the tag/header/status code, or `file:line` — never a grade on a hunch
(rubric → Conservatism rule).

## Phase 5 — Work Plan

Turn every 🔴 and 🟡 into an ordered task: blockers first, then gaps by how much of the site
they affect (a template-wide issue outranks a one-page issue). Each task names the concrete
fix and cites its evidence. `references/output-format.md` has the exact table shape.

## Phase 6 — Write the report

1. Resolve the report directory: `config.reportDir` if set, else `docs/seo-audits/` at the
   audited repo's root. **Create the directory if it doesn't exist** — the one
   directory-creation action this otherwise read-only skill takes by design, the same
   pattern `audit-hipaa` uses for `docs/hipaa-audits/`.
2. Filename: `YYYY-MM-DD-<slug>.md`, today's date and a short kebab-case slug describing the
   scope (`2026-09-12-homepage-and-blog.md`, `2026-09-12-full-site.md`).
3. **Never overwrite an existing dated report.** Each audit is its own point-in-time record
   of a moving target — the deployed site and how search engines currently treat it — so a
   same-day re-run with the same slug appends a numeric suffix (`-2`, `-3`) rather than
   replacing the earlier file. See `docs/decision-records/2026-09-12--audit-seo-reports-are-dated.md`.
4. Fill `assets/audit-report.md`'s shape (`references/output-format.md`).
5. Print the whole report in chat, then save the identical text. This is the **only** file
   the audit writes — everything else stays read-only unless the user explicitly asks for
   the findings to be applied (and even then, the Work Plan describes the fix; applying it
   is a separate, explicit ask, not something this skill does by default).

## Re-running after fixes

A re-audit is a normal, expected use of this skill — SEO fixes should be verified, and the
dated-file convention exists so a project accumulates a trail of these snapshots rather than
erasing the evidence of what changed. When re-running: use a new dated file (never edit an
old one), and if the user wants a before/after comparison, diff the two dated reports rather
than trying to reconcile them into one.

## Allowed vs. forbidden actions

**Allowed:**
- Read-only HTTP requests (GET/HEAD) to the site under audit, serially, within
  `crawl.maxPages`, respecting `robots.txt`.
- A browser fetch (claude-in-chrome) of the same bounded sample, to compare rendered DOM
  against raw HTML for dimension 6.
- Reading repo source: metadata exports, routing config, `robots.txt`/sitemap generators,
  `next.config.js`/`astro.config.mjs`/etc.
- Running the project's own lint/typecheck/test if that's useful to confirm a code-mode
  finding (e.g. confirming a metadata export actually type-checks) — never setup, install,
  migrate, or deploy commands.

**Forbidden:**
- Building the project to inspect its output (`next build`, `astro build`, …) — that writes
  `.next/`/`dist/`/build artifacts, a second write this skill doesn't have. Resolve the
  question against the live URL instead, or mark it ⚪.
- Crawling beyond the bounded sample, or crawling a site the user hasn't said they own or
  are engaged to audit.
- Any request that could be read as an attack surface probe (parameter fuzzing, auth
  bypass attempts, load testing) — this is an SEO audit, not a penetration test.
- Editing any file, committing, or pushing. The report is the only write.
