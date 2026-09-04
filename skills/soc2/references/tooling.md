# SOC 2 — Tooling Reference

> This skill defaults to a DIY approach: no assumption that a compliance platform (Vanta, Drata,
> Secureframe, or similar) is in use. This file covers what native, already-owned tools can cover
> on their own, then closes with an honest note on when a platform starts paying for itself.

## Table of Contents

1. [The DIY stack](#the-diy-stack)
2. [What each layer covers](#what-each-layer-covers)
3. [When a compliance platform starts paying for itself](#when-a-compliance-platform-starts-paying-for-itself)

---

## The DIY stack

Most of what a compliance platform automates can be assembled from tools an organization already
has, plus a small number of spreadsheets:

| Layer | Tool | Covers |
|---|---|---|
| Identity | SSO/IdP already in use (Google Workspace, Okta, Microsoft Entra, or a cloud provider's own IAM) | Provisioning, deprovisioning, MFA enforcement, access review exports |
| Cloud audit trail | AWS CloudTrail / GCP Cloud Audit Logs / Azure Monitor | CC7 logging and monitoring evidence |
| Source control & CI | GitHub/GitLab + branch protection + required CI checks | CC8 change management evidence |
| Vulnerability scanning | Dependabot/Snyk/`npm audit` or equivalent, plus a container/infra scanner | CC7 vulnerability management |
| Ticketing | Whatever the team already uses (Linear, Jira, GitHub Issues) for onboarding/offboarding and incident tracking | CC6 provisioning records, CC7 incident records |
| Evidence storage & tracking | A version-controlled evidence repo or a well-organized shared drive, plus the gap matrix spreadsheet (`assets/gap-matrix.csv`) | The full evidence-collection layer described in `evidence-and-monitoring.md` |
| Policy authoring & versioning | Plain markdown or docs tool with version history (this skill's `assets/policies/` skeletons are the starting point) | CC1–CC3 governance documentation |

None of this requires a subscription beyond what the org is very likely already paying for.

A minimal evidence-repo layout that ties the stack together without any platform:

```
evidence/
  2026-Q1/
    access-review/
      production-aws-iam-export.csv
      access-review-signoff.pdf
    change-management/
      sampled-prs-2026-q1.csv        # from `gh pr list --state merged --search "merged:2026-01-01..2026-03-31"`
    vulnerability-management/
      dependabot-alerts-export.csv
      remediation-tracker.csv
  2026-Q2/
    ...
policies/
  information-security-policy.md      # version-controlled alongside the code
  ...
```

---

## What each layer covers

- **Identity/SSO** is the highest-leverage layer — get this right and CC6 (the criterion most
  heavily tested) is largely covered by exports from a single system rather than assembled
  piecemeal.
- **Cloud audit trail + CI** together cover the bulk of CC7/CC8 evidence, since most relevant
  events (deploys, infrastructure changes, access changes) already generate a log entry
  somewhere — the work is making sure retention is long enough and someone reviews it
  periodically, not building new logging from scratch.
- **The gap matrix and evidence repo** are the connective tissue — without a platform tracking
  control status automatically, a maintained spreadsheet and a disciplined "capture as you go"
  habit (see `evidence-and-monitoring.md`) is what keeps evidence from being reconstructed from
  memory right before fieldwork.

---

## When a compliance platform starts paying for itself

Be honest about the trade-off rather than dismissing platforms outright. A platform (Vanta,
Drata, Secureframe, or similar) is usually worth it when:

- **This will repeat annually and the team is small.** The ongoing maintenance burden of
  DIY evidence collection — chasing quarterly access reviews, re-checking control status,
  re-running vendor assessments — is exactly what these platforms automate via integrations, and
  the time saved compounds every year.
- **Multiple frameworks are in scope.** An organization pursuing SOC 2 alongside ISO 27001, HIPAA,
  or GDPR gets real leverage from a platform's shared control mapping across frameworks — DIY
  tracking of overlapping requirements across multiple spreadsheets gets error-prone fast.
  Recommendation not to add optional Trust Services categories unless required (see
  `trust-services-criteria.md`) is independent of this — multi-framework leverage is about
  entirely separate compliance regimes, not SOC 2 categories.
- **There's no dedicated compliance owner.** Platforms provide guardrails and reminders (e.g. "an
  access review is due") that substitute for a person whose job includes tracking this
  proactively.

Stay DIY when the deadline is near-term, this is a one-time or first-time effort, the team is
small enough that a maintained spreadsheet is genuinely manageable, and budget is a real
constraint — the DIY stack above, done consistently, produces the same evidence a platform
would generate, just without automation.
