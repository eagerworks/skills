---
name: audit-seo
description: >-
  Audits a website's technical SEO — crawlability and indexation, on-page metadata, content and heading semantics, structured data, URL architecture and internationalization, JavaScript rendering, performance and Core Web Vitals signals, and measurement setup — against a live URL, a repo's source, or both, and returns a graded report with a mechanical verdict and an ordered Work Plan. Use when asked "audit the SEO of this site", "why isn't this page ranking", "review our meta tags", "check our sitemap and robots.txt", "is this site indexable", "SEO health check", "technical SEO review", "audit the SEO of this repo/codebase", or to re-check a site after SEO fixes were made. Never fabricates search volume, rankings, or traffic — those are marked unverifiable with the tool a human should use instead. The report is printed in chat and saved to a dated file under docs/seo-audits/.
metadata:
  author: eagerworks
  version: "1.0.0"
---

# Audit SEO Skill

**Technical SEO** is everything that determines whether a search engine can find, render,
and correctly understand a page — as distinct from keyword strategy, content marketing, or
link building, none of which this skill touches. This skill audits a site (or the repo that
builds it, or both) across eight dimensions and produces the **ordered work plan** that
closes the gaps it finds.

The audit is **read-only with exactly one write**: the finished report is printed in chat
**and** saved to a dated file under `docs/seo-audits/` at the audited repo's root
(`docs/seo-audits/` is created if missing; each run gets its own dated file — see gotcha 1).
Nothing else is created, edited, committed, or pushed.

## Discovery — Do This First

**1. Resolve the mode** from what you were actually given:

| Given | Mode | What gets graded |
|---|---|---|
| A URL only | **Live** | Served HTML, headers, `robots.txt`, `sitemap.xml`, rendered DOM |
| A repo only, no reachable URL | **Code** | Metadata source, routing, sitemap/robots generators — runtime-only checks marked ⚪ |
| Both | **Combined** | Live evidence, traced back to the source line that causes it |

Don't assume a repo and a URL are the same site just because they're both in front of you —
confirm the URL actually serves that repo's output first (gotcha 4).

**2. In Code or Combined mode, detect the framework** — where SEO metadata lives differs
enough between a Next.js app router, a Rails view, and a static site generator that guessing
produces wrong findings. Full signal table: `references/framework-detection.md`.

**3. Read `.eagerworks/audit-seo.json` if present** (`references/config.md`) — it can move
the report directory, name explicit sample URLs, bound the crawl, disable a dimension, or
pre-declare a fact like Search Console verification.

**4. Build a bounded page sample** — the homepage plus one page per distinct template, capped
at `crawl.maxPages` (default 20). Never the whole site (gotcha 5).

## What This Skill Does NOT Do

It doesn't do keyword research, write content, build backlinks, or produce search volume,
ranking position, traffic, or competitor data — those need tools this skill has no access to
(Search Console, a rank tracker, analytics) and are always graded ⚪ with the tool named,
never estimated (gotcha 2). It doesn't fix anything — the Work Plan describes changes, it
doesn't apply them. It doesn't crawl an entire site, and it never recommends a black-hat
tactic (gotcha 9).

## The Eight Dimensions

| # | Dimension | The site needs… |
|---|---|---|
| 1 | Crawlability & indexation | `robots.txt`, meta robots / `X-Robots-Tag`, sitemap, canonical, correct status codes, short redirect chains |
| 2 | On-page metadata | Unique, non-truncated titles & descriptions, Open Graph / Twitter cards |
| 3 | Content & heading semantics | One `<h1>`, an ordered heading hierarchy, real content, image `alt` text, descriptive internal links |
| 4 | Structured data | Valid JSON-LD, a correct `@type`, required properties present, no markup/content mismatch |
| 5 | Architecture, URLs & i18n | Readable URLs, consistent trailing-slash/host, HTTPS, correct `hreflang` where multi-language |
| 6 | Rendering & JavaScript SEO | Content present in the **raw** HTML, not only after hydration; no client-side soft 404s |
| 7 | Performance, Core Web Vitals & mobile | Render-blocking resources minimized, images sized, caching headers, mobile viewport |
| 8 | Measurement | Search Console and analytics installed and verifiable — the site can prove its own SEO |

Full checks, decision rules, and evidence for each: `references/rubric.md` — read it before
grading, it is the authoritative checklist.

## Grades and Verdict

| Grade | Meaning |
|---|---|
| 🔴 **Blocker** | Actively prevents indexation or correct representation (`noindex` in production, `Disallow: /`, malformed JSON-LD, JS-only critical content) |
| 🟡 **Gap** | Indexable but weaker than it should be (thin metadata, missing structured-data properties, no caching headers) |
| 🟢 **Pass** | Checked against a concrete rule and satisfied |
| ⚪ **Unverifiable** | Needs a tool this skill doesn't have (Search Console, a rank tracker, field-data performance) or a live URL when only a repo was given |

Verdict: **Blocked** (any 🔴) → **At risk** (🟡 only) → **Healthy**. The report's centrepiece
is the **Work Plan**: every 🔴 and 🟡 turned into an ordered task with effort (S/M/L),
evidence (a URL + tag/header, or `file:line`), and the concrete fix. Format:
`references/output-format.md`.

## Reference Files (read these on demand)

| Task | Read |
|---|---|
| The eight dimensions in full, grade rules, conservatism rule | `references/rubric.md` |
| Running the audit end-to-end; allowed vs. forbidden actions; writing the dated report | `references/audit-workflow.md` |
| The exact report markdown and the Work Plan table | `references/output-format.md` |
| How to fetch raw HTML vs. rendered DOM safely, sampling limits, and what genuinely can't be measured this way | `references/live-site-checks.md` |
| Where SEO metadata lives per framework (Next.js, Astro, Nuxt, SvelteKit, Rails, static-site generators, WordPress, …) | `references/framework-detection.md` |
| The optional `.eagerworks/audit-seo.json` config | `references/config.md` |

Copyable assets live in `assets/`:
- `assets/audit-report.md` — the report template; fill it in and save it as a dated file
  under `docs/seo-audits/`
- `assets/structured-data.examples.md` — valid JSON-LD starters for the types dimension 4
  checks most often
- `assets/audit-seo.example.json` — starter config

## Critical Gotchas

1. **One write, and it's dated, not overwritten.** The only file you create is the dated
   report under `docs/seo-audits/` (or `reportDir` from config) — never a meta tag, never a
   commit. Unlike most audit skills in this collection, don't overwrite an existing report:
   an SEO audit is a snapshot of a moving target (the deployed site, and how search engines
   currently treat it), and the fix→re-audit trail is itself the deliverable
   (`docs/decision-records/2026-09-12--audit-seo-reports-are-dated.md`). Same-day re-run,
   same slug → append a numeric suffix.

2. **Never invent a number you didn't measure.** Search volume, ranking position, traffic,
   competitor comparisons, and even a specific Core Web Vitals score are not things a fetch
   can produce. Every one of them is ⚪, naming the tool a human should open (Search Console,
   a rank tracker, Lighthouse/PageSpeed Insights) — even when the user asks directly for a
   number (`docs/decision-records/2026-09-12--audit-seo-never-fabricates-search-data.md`).

3. **Raw HTML and the rendered DOM are two different fetches — get both.** A crawler that
   doesn't execute JavaScript sees the raw response; a person sees the rendered page.
   Fetching only one and assuming the other matches is exactly how a JS-SEO blocker gets
   missed. `references/live-site-checks.md` has the recipe.

4. **A `noindex` on staging is not a production finding.** Confirm which environment a URL
   actually serves — a preview/staging deploy correctly blocking search engines is not the
   same as production doing it — before grading it 🔴.

5. **Never crawl the whole site.** A bounded sample (default 20 pages, `crawl.maxPages`),
   serial requests with a delay, `robots.txt` respected regardless of config, and only a
   site the user has said they own or are engaged to audit.

6. **Never build the project to see rendered output.** `next build`, `astro build`, etc.
   write build artifacts — a second write this skill doesn't have. Resolve the question
   against the live URL instead, or mark it ⚪ in Code-only mode.

7. **Core Web Vitals need field data a fetch can't produce.** You can flag *likely causes* —
   a render-blocking script, an unsized image — from a fetch. You cannot report an actual
   LCP/INP/CLS number that way. Say explicitly which is which.

8. **Zero findings is a valid, complete result.** Never pad the Work Plan on a healthy site.

9. **Never recommend a black-hat tactic** — link buying, PBNs, cloaking, doorway pages,
   hidden text — and flag one as 🔴 if the site already does it.

10. **A grade needs evidence.** A URL plus the exact tag/header/status code observed, or
    `file:line` in Code mode. "Probably missing a canonical" is not a finding.

11. **Print the whole report in chat, then save the identical text.** The user gets both;
    "see the file" is not the deliverable.
