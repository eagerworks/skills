# SOC 2 — Evidence & Monitoring Reference

> DIY evidence collection: what artifact an auditor will actually request for each control area,
> how to produce it without a compliance platform, and the recurring-control calendar that a
> Type II window runs on. See `references/tooling.md` for which native tools can generate each
> artifact.

## Table of Contents

1. [The general rule: capture as you go](#the-general-rule-capture-as-you-go)
2. [Evidence by control area](#evidence-by-control-area)
3. [Screenshot and export hygiene](#screenshot-and-export-hygiene)
4. [The recurring-control calendar](#the-recurring-control-calendar)
5. [Sampling and population completeness](#sampling-and-population-completeness)

---

## The general rule: capture as you go

The most common DIY mistake is treating evidence collection as something done at the *end* of
the observation window. By then, most of it is unrecoverable — a Type II auditor needs proof a
control operated *throughout* the period, and a screenshot taken the week before fieldwork only
proves the control exists *now*. Capture evidence on the same cadence the control itself runs:
export the access review the day it's done, not months later from memory.

A simple, durable pattern without any platform: a dated folder (or a dated set of files in a
version-controlled evidence repo) per month, one subfolder per control area, named artifacts
inside. This alone is often what a compliance platform is automating — see `tooling.md` for the
full layout.

```bash
# Example: capture a quarter's sampled, reviewed PRs as evidence the moment the quarter closes
gh pr list --repo your-org/your-app --state merged \
  --search "merged:2026-01-01..2026-03-31 base:main" \
  --json number,title,mergedAt,reviews \
  > evidence/2026-Q1/change-management/sampled-prs.json
```

---

## Evidence by control area

| Control area | What the auditor asks for | How to produce it without a platform |
|---|---|---|
| Access provisioning/deprovisioning | Ticket or record for each hire/termination showing access granted/revoked, with dates | Export from the ticketing system (or HRIS offboarding checklist) filtered to access-related tasks, one export per sampled period |
| Access reviews | Dated export of who-has-access-to-what, with sign-off from the resource owner | Pull an IAM/SSO group membership export each quarter, put it in a spreadsheet, have the owner mark each row reviewed and sign/date it |
| MFA enforcement | Proof MFA is required, not optional, for in-scope systems | Screenshot or export of the SSO provider's MFA enforcement policy setting |
| Change management | Sampled PRs/deploys showing review before merge | Export or link a sample of merged PRs to production branches, showing at least one approval before merge — most Git hosts support this via the API or a simple report |
| Vulnerability management | Scan results and remediation timelines matching the documented SLA | Export scan results (CI security tool, container scanner) periodically; track remediation dates against the SLA in a spreadsheet |
| Logging/monitoring | Proof logs are retained and alerts fire | Screenshot of retention configuration; a sampled alert with its response/resolution record |
| Backups | Proof backups run and are periodically restore-tested | Backup job success logs/exports; a dated record of the restore test and its result |
| Incident response | Records of any incidents (or a clean record) and how the documented process was followed | A log of incidents (even "none this period") plus, for any that occurred, the response timeline against the incident response plan |
| Security training | Completion records | Export from whatever training tool is used (even a shared spreadsheet with completion dates is acceptable) |
| Vendor risk | Evidence vendors were assessed before onboarding and periodically after | A vendor register (spreadsheet) noting assessment date, subservice org report reviewed (if applicable), and renewal/re-review date |

---

## Screenshot and export hygiene

- **Timestamp everything** — a screenshot with no visible date is weak evidence; prefer exports
  that include a generation timestamp, or annotate screenshots with the date captured.
- **Capture the filter/scope**, not just the result — e.g. an access review export should show
  which system/group it covers, not just a bare list of names.
- **Keep raw exports, not just summaries** — an auditor may want to trace a summary claim back to
  the underlying data; store both.
- **Consistent naming** — `2026-Q1_access-review_production-aws.csv` is far easier to hand to an
  auditor during fieldwork than an ad hoc collection of files named `Screenshot 2026-03-14.png`.

---

## The recurring-control calendar

The controls that generate ongoing evidence during a Type II window, and their typical cadence:

| Control | Cadence |
|---|---|
| Access review | Quarterly |
| Vulnerability/dependency scan review | Continuous (CI) with a periodic (e.g. monthly) summary review |
| Security awareness training | Annual (at onboarding + yearly refresh) |
| Risk assessment | Annual |
| Penetration test | Annual |
| BCDR/backup restore test | Annual |
| Policy review | Annual |
| Vendor/subservice org re-assessment | Annual, or on contract renewal |

Put these on a shared calendar with an owner per item before the observation window starts —
missing one recurring control mid-window is a common, avoidable exception.

---

## Sampling and population completeness

Auditors typically test a **sample**, not every instance, of a recurring control — but the
*population* the sample is drawn from must be complete and demonstrable. Practically: be able to
produce, for example, the full list of everyone who joined or left during the window (not just
the ones remembered), so the auditor's sample is drawn from a real, complete population rather
than a curated list. An HRIS export or a maintained roster is the simplest way to guarantee this.
