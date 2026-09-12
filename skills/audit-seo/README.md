# audit-seo

A portable agent skill that audits a website's **technical SEO** — whether search engines
can crawl, index, and correctly understand its pages — against a live URL, the repo that
builds it, or both, and returns the ordered list of work needed to fix what's wrong. Works
with Claude Code, Cursor, GitHub Copilot, Codex, Amp, and any agentic coding tool that can
read markdown files.

## What it covers

- Auto-detected scope: a live URL (**Live** mode), a repo with no reachable URL (**Code**
  mode), or both (**Combined** mode, tracing a live finding back to the source line that
  causes it)
- Eight dimensions: crawlability & indexation, on-page metadata, content & heading
  semantics, structured data, architecture/URLs/i18n, rendering & JavaScript SEO,
  performance & Core Web Vitals signals, and measurement (Search Console/analytics)
- A four-level grade per check — 🔴 Blocker / 🟡 Gap / 🟢 Pass / ⚪ Unverifiable — rolled up
  to a mechanical verdict: Blocked / At risk / Healthy
- A **Work Plan**: every blocker and gap as an ordered task with effort (S/M/L), evidence,
  and the concrete fix
- Never fabricates search volume, ranking position, traffic, or competitor data — those are
  always ⚪ with the tool a human should use instead (Search Console, a rank tracker,
  Lighthouse/PageSpeed Insights), even when asked directly
- Safe execution rules: a bounded, rate-limited page sample; `robots.txt` always respected;
  never builds the project to inspect output; never crawls beyond the sample
- The report is printed in chat **and** saved to a dated file under `docs/seo-audits/` in
  the audited repo (the directory is created if needed) — unlike most audit skills in this
  collection, the file is never overwritten, so a project accumulates a dated trail across
  fix→re-audit cycles. This is the one file the skill writes; it never commits or pushes
- Copyable, spec-valid JSON-LD starters for the structured-data types most sites need

## Layout

```
SKILL.md                          # hub: discovery, mode resolution, the eight dimensions, grades, gotchas (agent entrypoint)
references/
  rubric.md                       # full checklist per dimension, grade ladder, conservatism rule
  audit-workflow.md               # phase-by-phase procedure, allowed vs. forbidden actions, saving the dated report
  output-format.md                # the report markdown: scorecard, Work Plan, findings, footer
  live-site-checks.md             # raw HTML vs. rendered DOM, sampling limits, what genuinely can't be measured
  framework-detection.md          # where SEO metadata lives per stack (Next.js, Astro, Nuxt, Rails, static-site generators, WordPress, …)
  config.md                       # .eagerworks/audit-seo.json schema
assets/
  audit-report.md                 # report template → docs/seo-audits/YYYY-MM-DD-<slug>.md
  structured-data.examples.md     # valid JSON-LD for Organization, WebSite, BreadcrumbList, Article, Product
  audit-seo.example.json          # starter config
```

The agent loads [`SKILL.md`](SKILL.md) up front and opens the matching
[`references/`](references/) file on demand, so the entrypoint stays lean while the full
knowledge base is always available.

## Configuration

Zero configuration required. To move the report directory, name explicit sample URLs, bound
the crawl, disable a dimension that doesn't apply, or pre-declare a fact like Search Console
verification, add `.eagerworks/audit-seo.json` — see
[`references/config.md`](references/config.md) and
[`assets/audit-seo.example.json`](assets/audit-seo.example.json). Disabled dimensions are
always disclosed in the report footer.

## Install

See the [collection README](../../README.md#install). In short:

```bash
npx skills add eagerworks/skills --skill audit-seo
```
