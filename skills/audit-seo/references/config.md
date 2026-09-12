# Audit SEO — Configuration

`.eagerworks/audit-seo.json`, at the audited repo's root, is **entirely optional**. The
audit works with no config — it auto-detects the mode, samples a bounded set of pages, and
writes a dated report to `docs/seo-audits/`. Add the file only to move the report directory,
name explicit sample URLs instead of auto-sampling, change the crawl bounds, disable a
dimension that genuinely doesn't apply (e.g. `internationalization` on a single-language
site), or pre-declare a fact like Search Console verification.

## Resolution order

`.eagerworks/audit-seo.json` → statements in `AGENTS.md`/`CLAUDE.md` → the skill's built-in
defaults. A later source only fills in what an earlier one didn't set.

## Schema

All fields optional.

```jsonc
{
  // Where dated reports are written, relative to the repo root.
  "reportDir": "docs/seo-audits",

  "site": {
    // Used to resolve the site when only a repo is given (Code/Combined mode),
    // and to normalize the canonical-host check (5.3, 1.7).
    "baseUrl": "https://your.domain.com",

    // Skip auto-sampling and audit exactly these URLs (relative to baseUrl, or
    // absolute). Still capped by crawl.maxPages.
    "sampleUrls": ["/", "/pricing", "/blog/a-representative-post"]
  },

  "crawl": {
    // Hard cap on pages fetched in one run.
    "maxPages": 20,

    // Delay between requests, in milliseconds — serial, polite crawling.
    "requestDelayMs": 1000,

    // Cannot be set to false — see Rules below. Present so a repo can see the
    // default explicitly rather than needing to know it exists.
    "respectRobotsTxt": true
  },

  // Report language. Never inferred from the conversation's language.
  "language": "en",

  // Disable a dimension that does not apply (e.g. no multi-language content).
  // Disabled dimensions are always disclosed in the report footer.
  "dimensions": {
    "crawlability": { "enabled": true },
    "metadata": { "enabled": true },
    "content": { "enabled": true },
    "structuredData": { "enabled": true },
    "architecture": { "enabled": true },
    "rendering": { "enabled": true },
    "performance": { "enabled": true },
    "measurement": { "enabled": true }
  },

  // Facts you already hold, that turn a ⚪ into a 🟢 without asking again.
  // Input, not evidence — see Rules below.
  "known": {
    "searchConsoleVerified": true,
    "analyticsInstalled": "GA4, verified in the raw HTML"
  }
}
```

## Rules

- `crawl.respectRobotsTxt` is honoured regardless of what the file sets it to — this audit
  never fetches a path `robots.txt` disallows. The key exists in the schema so a repo can
  see the default is on, not so it can be turned off.
- `known.*` is input the skill takes on trust, not evidence it verified — it's disclosed in
  the footer exactly like a disabled dimension, so the reader knows which 🟢s came from a
  check versus a declaration.
- A disabled dimension appears in the footer as `dimensions disabled by config: <name>` and
  is excluded from the Scorecard and the verdict.
- `crawl.maxPages` bounds how much the audit fetches; it never removes the obligation to say,
  in the footer, how many pages exist versus how many were sampled when that's knowable
  (e.g. from the sitemap's URL count).

## Starter

Copy `assets/audit-seo.example.json` to `.eagerworks/audit-seo.json` and delete what you
don't need.
