# Changelog

All notable changes to this skill are documented here. Versions match `metadata.version` in `SKILL.md`'s frontmatter. When a change adds, renames, or removes a config key, or requires creating a new file in the consuming repo, that's called out under **Config** so an upgrade doesn't need to be reverse-engineered from the diff.

## [1.0.0] - 2026-09-12

Initial release.

### Added
- Audits a website's technical SEO across eight dimensions — crawlability & indexation, on-page metadata, content & heading semantics, structured data, architecture/URLs/i18n, rendering & JavaScript SEO, performance & Core Web Vitals signals, and measurement — graded 🔴 Blocker / 🟡 Gap / 🟢 Pass / ⚪ Unverifiable, rolled up to a mechanical verdict and an ordered Work Plan.
- Auto-detects scope: a live URL, a repo with no reachable URL, or both, tracing a live finding back to the source line that causes it in Combined mode.
- Never fabricates search volume, ranking position, traffic, or competitor data — those checks are always ⚪ with the tool a human should use instead (`docs/decision-records/2026-09-12--audit-seo-never-fabricates-search-data.md`).
- Exactly one write: the report is printed in chat and saved to a **dated** file under `docs/seo-audits/` in the audited repo (directory created if missing, never overwritten across runs — `docs/decision-records/2026-09-12--audit-seo-reports-are-dated.md`).
- May run a bounded, rate-limited fetch of a page sample and, in Code mode, the project's own lint/typecheck; never builds the project, crawls beyond the sample, or ignores `robots.txt`.

### Config
- New optional `.eagerworks/audit-seo.json` — `reportDir`, `site.{baseUrl,sampleUrls}`, `crawl.{maxPages,requestDelayMs,respectRobotsTxt}`, `language`, `dimensions.*.enabled`, `known.*`. See `references/config.md`.
