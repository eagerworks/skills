# `audit-seo` never fabricates search volume, rankings, or traffic — those checks are always ⚪

- **Date:** 2026-09-12

## Context

An SEO audit's most-requested output, in practice, is often the thing this skill has no way
to actually produce: "how much search traffic could we gain", "what's our current ranking
for X", "how do we compare to this competitor". These numbers are exactly the kind of
plausible-sounding, unverifiable-by-the-reader output a language model can produce
confidently and wrong — a fetch of a site's HTML and headers contains no information about
how many people search for a term, where a page currently ranks, or what a competitor's
traffic looks like. Those require a signed-in Search Console session, a keyword-planning
tool, or a third-party index (Ahrefs, Semrush, Moz) this skill has no access to.

The risk is specific to this domain in a way it isn't for the collection's other audit
skills: `loop-engineering-audit` and `repo-handoff` grade things the repo itself contains
(a test command, a lockfile), so there's little room for a plausible-but-wrong number to
slip in. An SEO audit sits one step away from a genuinely data-driven discipline (rank
tracking, traffic analytics) that a client or stakeholder will recognize by its *numbers*,
not its structure — making a fabricated "estimated 12,000 monthly searches" or "you likely
rank #4" far more dangerous: it reads as authoritative, it's expensive to independently
verify, and if repeated into a client deliverable it damages trust in the entire report the
moment it's checked against a real tool.

Two alternatives were considered and rejected:

1. **Let the skill web-search for the number and cite a source.** Rejected — a web search
   can surface a stale, wrong, or unrelated result (a different domain's traffic estimate, a
   ranking snapshot from months ago), and presenting it inside a graded audit report implies
   a confidence the underlying search doesn't have. The skill's rubric grades observed
   evidence (a tag, a header, a status code); a number from an ungraded web search doesn't
   belong at the same confidence level.
2. **Produce a labeled "rough estimate" instead of refusing.** Rejected — a labeled estimate
   still gets copy-pasted into a deliverable without its label, and "rough estimate" doesn't
   change the fact that the skill has no actual signal to base it on. The honest answer is
   that this skill can't measure it, not a hedge that sounds like it can.

## Decision

`audit-seo` never produces a number for search volume, ranking position, traffic, or a
competitor comparison — regardless of whether the user asks directly. Every such request is
graded ⚪ **Unverifiable** and paired with the exact tool that would actually answer it
(Google Search Console, Keyword Planner, a rank tracker, GA4/Plausible, Ahrefs/Semrush/Moz).
This extends to Core Web Vitals: a fetch can flag *likely causes* (a render-blocking script,
an unsized image) but never an actual LCP/INP/CLS number, which needs field data or a real
timed measurement tool (`skills/audit-seo/references/live-site-checks.md` → "What is
genuinely not measurable this way").

This is enforced at the rubric level (`references/rubric.md` → Conservatism rule) and the
output-format level (`references/output-format.md` → Rules), not left as a suggestion in
`SKILL.md` alone.

## Consequences

- A user who wants these numbers gets redirected to the right tool instead of a number that
  looks like an answer — the report is honest about a real gap in what this skill can see,
  rather than papering over it.
- The skill's Work Plan and Scorecard stay strictly evidence-based, which keeps its outputs
  falsifiable and its grades trustworthy — a reader can independently re-check any 🔴/🟡/🟢 by
  fetching the same URL or reading the same source line, which is not true of an invented
  estimate.
- This makes `audit-seo` narrower than a "full SEO audit" a marketing agency might sell —
  that's a deliberate scope boundary, not an oversight; `SKILL.md` → "What This Skill Does
  NOT Do" states it plainly so a user isn't surprised mid-report.

## Related

- [2026-09-12--audit-seo-reports-are-dated](2026-09-12--audit-seo-reports-are-dated.md) —
  the sibling decision governing this skill's other distinguishing behavior.
- [2026-08-21--ignore-paths-must-be-disclosed](2026-08-21--ignore-paths-must-be-disclosed.md) —
  the never-silently-skip precedent this decision extends to unmeasurable data, not just
  config-excluded paths.
- `skills/audit-seo/references/rubric.md` → "Conservatism rule" and
  `skills/audit-seo/references/live-site-checks.md` → "What is genuinely not measurable this
  way" — where this decision is operationalized.
