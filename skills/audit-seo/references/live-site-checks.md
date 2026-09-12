# Audit SEO — Live Site Checks

How to gather evidence from a real URL safely, and — just as important — an honest list of
what this skill genuinely cannot measure that way.

## Raw HTML vs. rendered DOM

Dimension 6 (`references/rubric.md`) exists because a page can look complete to a person in
a browser while a crawler — which by default does not execute JavaScript the way a browser
does — sees an empty shell. Get **both** responses for every sampled URL:

1. **Raw HTML** — a plain HTTP GET with no JavaScript execution (`curl`, or the WebFetch
   tool). This is close to what a crawler that doesn't render JS sees.
2. **Rendered DOM** — the same URL loaded in a real browser (claude-in-chrome), after it's
   finished loading, read via the page-reading tool.

Diff them for the page's actual content: title, main heading, body text, primary internal
links. If the rendered version has substantial content the raw version never had, that's a
6.1/6.2 finding — cite both snapshots as evidence, not just one.

```bash
# ✅ correct — raw HTML, no JS execution
curl -sL <url>

# then, separately, load <url> in claude-in-chrome and read the rendered page text
```

Don't skip step 1 just because the page renders correctly in a browser — the rendered view
is not evidence about what a non-executing crawler sees, and assuming SSR/SSG is working
because the framework supports it is exactly the mistake gotcha 3 warns against.

## `robots.txt`, sitemaps, and headers

```bash
curl -sI <url>                       # status code, redirect target, X-Robots-Tag, caching headers
curl -s <url>/robots.txt             # Disallow rules, sitemap reference
curl -s <url>/sitemap.xml            # validate as XML; spot-check a sample of its URLs
```

For redirect chains, follow manually rather than with `-L` alone so you can count hops:

```bash
curl -sI <url> | grep -i '^location:'   # repeat against the returned Location until it stops
```

## Sampling and rate limits

- Cap at `crawl.maxPages` (default 20) and space requests by `crawl.requestDelayMs` (default
  1000ms) — serial, not parallel. This is a courtesy to the site being audited, not a
  performance optimization; never lower it to make the audit faster.
- Respect `robots.txt` `Disallow` rules for the paths you'd otherwise sample — if the rules
  block a page you need to grade, note it as a limitation rather than fetching it anyway.
- Only audit a site the user has said they own, work on, or are engaged to review. A URL
  appearing in conversation isn't itself permission to crawl it at any depth.

## What is genuinely not measurable this way

Be explicit with the user about this list — it's the difference between an honest audit and
one that quietly makes things up:

- **Search volume, keyword difficulty, ranking position** — need Google Keyword Planner,
  Search Console, or a third-party rank tracker (Ahrefs, Semrush, etc.). Always ⚪, naming
  the tool.
- **Actual traffic** — needs analytics access (GA4, Plausible) this skill doesn't have.
- **Real Core Web Vitals (LCP, INP, CLS)** — these are field metrics aggregated from real
  users (Chrome UX Report) or a lab tool that actually runs a timed page load
  (Lighthouse/PageSpeed Insights). A fetch-based audit can flag *likely causes* — a
  render-blocking script, an unsized image — but cannot report a vitals number. Say which is
  which; never state a number you didn't get from one of those tools.
- **Backlink profile, domain authority, competitor comparisons** — need a third-party index
  this skill has no access to. ⚪, naming the tool (Ahrefs, Semrush, Moz).
- **Whether Search Console is actually verified** — a verification meta tag or DNS record
  can be *checked for*, but confirming verification succeeded needs a signed-in session.
  `known.searchConsoleVerified` in config lets a user assert this without re-asking every
  run (`references/config.md`).

If a user asks directly for one of these numbers, don't produce an estimate "to be helpful"
— name the tool that would actually answer it (`docs/decision-records/2026-09-12--audit-seo-never-fabricates-search-data.md`).
