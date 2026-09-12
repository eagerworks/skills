# `audit-seo`'s report is a dated file, following `audit-hipaa` rather than the fixed-name default

- **Date:** 2026-09-12

## Context

`2026-08-28--audit-report-saved-to-docs.md` established the repo-wide default for an audit
skill's report: a fixed filename at the audited repo's root, overwritten on every run, so
the project carries one current audit and `git diff` shows the delta. `loop-engineering-audit`
and `repo-handoff` both follow it. `audit-hipaa` is the one deliberate exception, justified
there by a legal retention requirement (§164.316(b)(2)(i)) that doesn't apply outside HIPAA.

`audit-seo` doesn't have a retention regulation to point to, but it has the same underlying
shape as `audit-hipaa`'s exception for a different reason: what it's auditing is not the
repo itself, but a **moving target the repo doesn't fully control** — the deployed site, and
how search engines currently treat it. Two facts follow from that:

1. **The site can change out from under the repo.** A CDN cache, a marketing team's CMS
   edit, a DNS change, or a third-party plugin update can all alter what a live audit finds
   without a single commit landing. An overwritten fixed-name report loses the ability to
   say "this is what changed, and when" for anything that happened outside the codebase —
   exactly the information `git diff` can't recover for this domain, unlike for a
   code-only audit like `loop-engineering-audit`.
2. **The intended workflow is audit → fix → re-audit, and the trail is the deliverable.**
   An agency or in-house SEO practitioner runs this skill, ships fixes, and re-runs it to
   prove the fixes worked — often for a client who wants to see the history, not just the
   latest snapshot. A single overwritten file destroys that evidence on the very next run.

Two alternatives were considered and rejected:

1. **Fixed name, overwritten, like `loop-engineering-audit`.** Consistent with the repo-wide
   default and simpler. Rejected because it erases the audit→fix→re-audit trail that makes
   this skill useful as a recurring practice rather than a one-time snapshot, and because
   `git diff` can't reconstruct what changed on the *live site* between two audits the way it
   can for a code-only report.
2. **A dated file, but only when a live URL was audited (Code-mode audits get the fixed
   name).** Rejected as unnecessary complexity — a Code-mode audit of a repo's SEO surface
   is just as plausibly re-run as fixes land in source, and a consumer of this skill
   shouldn't need to remember two different output conventions depending on which mode a
   given run happened to use.

## Decision

`audit-seo` always writes its completed report to `docs/seo-audits/YYYY-MM-DD-<slug>.md` in
the audited repo, creating `docs/seo-audits/` if it doesn't already exist — this is the
skill's one write, matching `audit-hipaa`'s pattern exactly
(`skills/audit-hipaa/references/audit-workflow.md` → "Phase 6 — Write the report";
`2026-08-28--hipaa-audit-report-is-a-dated-repo-file.md`).

The filename is never overwritten. A same-day re-run with the same slug appends a numeric
suffix (`-2`, `-3`) rather than replacing the earlier file.

## Consequences

- A repo using this skill accumulates a dated audit trail in `docs/seo-audits/` over time,
  which is itself useful for a team or client wanting to see SEO health tracked over time —
  the skill doesn't build any tooling around that trail (no index, no diffing), it's a plain
  side effect of the convention.
- Running the skill against a repo for the first time creates a new top-level directory as a
  side effect — `SKILL.md` and `references/audit-workflow.md` both call this out explicitly
  so it's never a silent surprise, the same discipline `audit-hipaa` applies.
- Two skills in this collection now use the dated-file pattern for two unrelated reasons
  (legal retention vs. a moving external target). A future reviewer shouldn't read this as
  the pattern loosening generally — the repo-wide default from `2026-08-28--audit-report-saved-to-docs.md`
  still applies to any future audit skill whose subject is the repo itself; a skill needs its
  own justification, not just precedent, to deviate from it.

## Related

- [2026-08-28--audit-report-saved-to-docs](2026-08-28--audit-report-saved-to-docs.md) — the
  repo-wide fixed-name-overwritten default this decision deviates from.
- [2026-08-28--hipaa-audit-report-is-a-dated-repo-file](2026-08-28--hipaa-audit-report-is-a-dated-repo-file.md) —
  the sibling exception this decision follows structurally.
- `skills/audit-seo/references/audit-workflow.md` → "Phase 6 — Write the report" — the
  procedure this ADR justifies.
