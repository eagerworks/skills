# Audit SEO — Rubric

The authoritative checklist. Each dimension lists concrete checks, what evidence satisfies
them, and the grade decision. Grade every check that applies to the resolved mode; roll up
each dimension to its worst check.

Grades: 🔴 **Blocker** · 🟡 **Gap** · 🟢 **Pass** · ⚪ **Unverifiable**. Definitions and the
verdict rule are at the end.

**Mode changes what evidence exists, not what the check means.** In **Live** mode, every
check is graded against the served response — HTML, headers, `robots.txt`, `sitemap.xml`.
In **Code** mode, grade against the source that generates those things, and mark anything
that only a running server could confirm (actual response headers, real redirect chains,
whether hydration matches server output) ⚪ rather than guessing. In **Combined** mode,
prefer live evidence and cite the source line that causes it.

## Dimension 1 — Crawlability & indexation

*Can a search engine reach this page and choose to index it?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 1.1 `robots.txt` exists and is valid | Fetches with 200, parses, doesn't block the whole site (`Disallow: /`) for the primary user-agent | 🔴 blocks the site in production; 🟡 malformed or missing (search engines assume everything is allowed, which is usually fine, but say so) |
| 1.2 No unintended `noindex` | No `<meta name="robots" content="noindex">` / `X-Robots-Tag: noindex` on a page meant to rank | 🔴 present on a production page meant to rank |
| 1.3 Sitemap exists and is referenced | `sitemap.xml` (or an index of them) is valid XML, listed in `robots.txt` or submitted to Search Console, and its URLs 200 | 🟡 missing; 🟡 contains 404s/redirects/noindexed URLs |
| 1.4 Canonical tag present and correct | Every indexable page has exactly one `<link rel="canonical">` pointing at the preferred URL (itself, normally) | 🔴 missing on duplicate-prone pages (faceted nav, `?utm_*`, pagination); 🟡 present but points at the wrong host/scheme |
| 1.5 Status codes are correct | Real pages 200, missing pages 404 (not 200 with "not found" text — a **soft 404**), moved pages 301 | 🔴 soft 404s on money pages; 🟡 a few, low-traffic |
| 1.6 Redirect chains are short | A redirect resolves in ≤ 1 hop | 🟡 2+ hops; 🔴 a loop |
| 1.7 HTTPS and one canonical host | Site resolves to exactly one of `https://www.` or `https://` apex, the other redirects (301) to it | 🔴 HTTP still serves content; 🟡 both host variants serve 200 with no redirect (duplicate content) |

Evidence: `curl -sI <url>` for status/headers, `curl -s <url>/robots.txt`, fetch and validate
`sitemap.xml` as XML, `curl -sI` following `-L` and counting hops for redirect checks. In
Code mode: grep the framework's `robots`/`sitemap`/`redirects` config (see
`references/framework-detection.md`) and mark checks that need a live response ⚪.

## Dimension 2 — On-page metadata

*Does every page tell a search result and a social share what it is?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 2.1 `<title>` present, unique, right length | 50–60 characters, one per page, distinct across the sampled pages | 🔴 missing or identical across pages (templated but unfilled); 🟡 truncated (>60 chars) or thin (<15 chars) |
| 2.2 Meta description present, unique | 140–160 characters, distinct per page, summarizes the actual content | 🟡 missing, duplicated across pages, or generic boilerplate |
| 2.3 Open Graph tags | `og:title`, `og:description`, `og:image`, `og:url` present and match the page | 🟡 missing (link previews degrade); 🟡 `og:image` 404s |
| 2.4 Twitter Card tags | `twitter:card` at minimum (`summary_large_image` when an image exists) | 🟡 missing |
| 2.5 `<html lang>` set | Matches the page's actual language | 🟡 missing or wrong |
| 2.6 Favicon and viewport meta | `<link rel="icon">` resolves; `<meta name="viewport" content="width=device-width, initial-scale=1">` present | 🟡 missing (mobile rendering / branding, not a ranking blocker) |

Evidence: parse the raw HTML `<head>` for the sampled pages; note titles/descriptions
side by side to catch duplication across templates.

## Dimension 3 — Content & heading semantics

*Does the page's structure tell a crawler what matters, and can everyone read it?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 3.1 Exactly one `<h1>` per page | One, describing the page's topic | 🟡 zero or multiple `<h1>`s |
| 3.2 Heading hierarchy is ordered | `h1` → `h2` → `h3` without skipping levels for styling | 🟡 headings chosen for font size, skipping levels |
| 3.3 Real content exists | The rendered page has substantive text, not a stub or a wall of boilerplate | 🔴 a target page is thin/empty in the raw HTML (see dimension 6 for the JS case) |
| 3.4 Images have meaningful `alt` | Content images carry descriptive `alt`; decorative images use `alt=""` | 🟡 missing `alt` on content images across a sample |
| 3.5 Internal links use descriptive anchor text | Anchor text describes the destination, not "click here"/bare URLs | 🟡 generic anchor text is the norm |
| 3.6 No broken internal links in the sample | Internal links in the sampled pages resolve (200/3xx to a real page) | 🟡 a broken internal link found; 🔴 primary nav/footer links broken |

Evidence: extract `<h1>`–`<h6>` in document order; `alt` attribute presence on `<img>`;
`href` targets fetched with a HEAD request (bounded to the sample, never the whole site —
gotcha 5).

## Dimension 4 — Structured data

*Can a search engine understand this content as an entity, not just text?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 4.1 JSON-LD is valid | `<script type="application/ld+json">` parses as JSON | 🔴 malformed JSON (the whole block is silently ignored) |
| 4.2 `@type` matches the content | E.g. `Article` on a blog post, `Product` on a product page, not a mismatched or overly generic type | 🟡 wrong or generic `@type` |
| 4.3 Required properties present | Google's minimum for that type (e.g. `Article` needs `headline`, `image`, `datePublished`) | 🟡 missing recommended properties; 🔴 missing a required one |
| 4.4 Markup matches visible content | Structured data doesn't claim a price/rating/date that isn't actually shown on the page | 🔴 mismatch (this is a Google spam policy violation, not just a quality gap) |
| 4.5 Sitewide entities present where relevant | `Organization`/`WebSite` (+ `SearchAction`) on the homepage; `BreadcrumbList` on deep pages | 🟡 missing |

Evidence: extract and `JSON.parse` every `ld+json` block; compare declared properties
against the visible DOM/text. Starter markup for common types:
`assets/structured-data.examples.md`.

## Dimension 5 — Architecture, URLs & internationalization

*Is the site's shape legible and consistent?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 5.1 URLs are readable and stable | Lowercase, hyphenated, human-readable slugs; no session IDs or unnecessary query params in canonical URLs | 🟡 opaque IDs, mixed case, or churny slugs |
| 5.2 URL depth is reasonable | Important pages reachable within ~3 clicks from the homepage | 🟡 a target page buried deeper with no internal links pointing at it |
| 5.3 Trailing-slash and case consistency | The site picks one convention and applies a redirect for the other, not two live variants | 🟡 both `/page` and `/page/` serve 200 with no canonical/redirect |
| 5.4 `hreflang` correct (multi-language sites only) | Every language variant lists every other variant plus itself (`x-default` where relevant), reciprocally | 🔴 one-directional or missing `hreflang` on a genuinely multi-locale site; N/A on a single-language site |
| 5.5 Pagination is crawlable | Paginated lists use real, linked URLs (not a JS-only "load more" with no URL state) — dimension 6 covers whether the content itself needs JS | 🟡 pagination has no crawlable URL |

Evidence: sample URL casing/slugs; for `hreflang`, fetch each declared locale variant and
check the reciprocal link exists.

## Dimension 6 — Rendering & JavaScript SEO

*Does the crawler see what the visitor sees?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 6.1 Primary content is in the raw HTML | The raw server response (no JS execution) contains the page's main content — title, body text, primary links | 🔴 the raw HTML is a near-empty shell (`<div id="root"></div>`) and content only appears after hydration |
| 6.2 Raw HTML and rendered DOM agree | Fetch the URL twice — once as a plain HTTP request, once through a real browser — and diff the two | 🟡 minor differences (a loading spinner replaced); 🔴 the rendered version adds content the raw version never had, for a page meant to rank |
| 6.3 No soft-404 via client routing | A client-side "not found" page still returns 200 at the HTTP layer with no server-side signal | 🟡 present (same failure as 1.5, called out separately because SPA routing causes it) |
| 6.4 Critical metadata isn't JS-only | `<title>`, canonical, meta description are present in the raw HTML, not injected client-side only | 🔴 metadata only appears after JS runs |

**This is the one dimension that requires two fetches, not one** — see gotcha 3 and
`references/live-site-checks.md` for exactly how to get the raw response and the rendered
DOM safely. In Code mode without a live URL, grade from the rendering strategy the
framework declares (SSR/SSG/ISR vs. client-only render) and mark 6.1–6.4 ⚪ with the
reasoning, never assume client rendering is broken just because the framework can do it.

## Dimension 7 — Performance, Core Web Vitals & mobile

*Does the page load fast enough, and does it work on a phone?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 7.1 Render-blocking resources are minimized | CSS/JS in `<head>` is small or deferred/async; no synchronous third-party scripts before content | 🟡 several blocking `<script>` tags with no `defer`/`async` before the fold |
| 7.2 Images are sized and modern-format | `width`/`height` (or `aspect-ratio`) set to prevent layout shift; `.webp`/`.avif` used or images aren't grossly oversized for their display size | 🟡 no dimensions (CLS risk); 🟡 large unoptimized images |
| 7.3 Caching headers on static assets | `Cache-Control` with a sane max-age on JS/CSS/images/fonts | 🟡 no caching headers on static assets |
| 7.4 Compression enabled | Response uses `gzip`/`br` (`Content-Encoding` header) | 🟡 uncompressed text responses |
| 7.5 Mobile viewport configured | Covered by 2.6; re-check here in the context of layout, not just presence | 🟡 present but content overflows viewport width in a rendered check |

**Say explicitly which of these are *measured* vs. *inferred*.** A fetch can observe
render-blocking scripts and missing image dimensions directly; it cannot report an actual
LCP/INP/CLS number — those need field data (gotcha 7). Never state a Core Web Vitals score
you didn't get from a real measurement tool.

## Dimension 8 — Measurement

*Can the site prove its own SEO, going forward?*

| Check | 🟢 when | 🔴 / 🟡 when |
|---|---|---|
| 8.1 Search Console verification present | A verification meta tag, DNS record, or `known.searchConsoleVerified: true` in config | ⚪ can't be confirmed from outside without a signed-in session; 🟡 no verification signal found and not asserted in config |
| 8.2 Analytics installed | A recognizable analytics snippet (GA4, Plausible, Fathom, etc.) present in the raw HTML | 🟡 none found |
| 8.3 Sitemap submitted / discoverable | `sitemap.xml` referenced from `robots.txt` (already checked in 1.3; cross-reference here for the measurement angle) | 🟡 exists but isn't discoverable by a crawler that only reads `robots.txt` |

This dimension is intentionally thin — it exists to flag when a site has **no way to find
out** whether the other seven dimensions are working in practice, not to grade traffic or
rankings (never available to this skill; see gotcha 2 and `references/live-site-checks.md`).

## Grade ladder

- 🔴 **Blocker** — actively prevents the page from being indexed or correctly represented
  (a `noindex` in production, a `Disallow: /`, a malformed JSON-LD, JS-only critical
  content). Any 🔴 ⇒ verdict **Blocked**.
- 🟡 **Gap** — the page is indexable but weaker than it should be (thin descriptions,
  missing structured data properties, no caching headers). Only 🟡 ⇒ **At risk**.
- 🟢 **Pass** — a concrete rule was checked and satisfied.
- ⚪ **Unverifiable** — needs a tool this skill doesn't have access to (Search Console,
  a rank tracker, real-user field data) or a live URL when only a repo was given. Always
  carries the exact tool or check a human should run; never changes the verdict.

## Conservatism rule

A grade other than 🟢 must cite evidence: a URL plus the exact tag/header/status code
observed, or a `file:line` in Code mode. If you can't produce that, the check is either 🟢
(you checked and it's fine) or ⚪ (you couldn't check) — never 🟡 on a hunch, and never 🔴
because a pattern merely looks risky. Never invent a number — search volume, ranking
position, traffic estimate, competitor comparison — that didn't come from an actual
measurement; those checks are always ⚪ with the tool named (`references/live-site-checks.md`
→ "What is genuinely not measurable this way").

Config `.eagerworks/audit-seo.json` may disable a dimension or bound the crawl — any
disabled dimension is disclosed in the report, never silently omitted
(`references/config.md`).
