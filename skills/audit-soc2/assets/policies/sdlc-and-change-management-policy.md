# SDLC & Change Management Policy

| | |
|---|---|
| **Owner** | [PLACEHOLDER: role, e.g. Head of Engineering] |
| **Approver** | [PLACEHOLDER: role] |
| **Review cadence** | Annual |
| **Last reviewed** | [PLACEHOLDER: date] |
| **Version** | [PLACEHOLDER: 1.0] |

> **Auditor note:** Auditors sample actual merged PRs/deploys against this policy. If the policy
> says review is required but sampled changes show unreviewed merges, that's an exception. See
> `references/technical-controls.md`.

## 1. Purpose

Defines how software and infrastructure changes are proposed, reviewed, tested, and deployed to
production.

## 2. Scope

Applies to all code and infrastructure changes affecting [PLACEHOLDER: system boundary].

## 3. Policy statements

- **Version control.** All code is maintained in [PLACEHOLDER: e.g. GitHub], with a complete
  history of changes.
- **Review before merge.** Changes to [PLACEHOLDER: production branch(es)] require at least
  [PLACEHOLDER: 1] approval from someone other than the author before merging. Enforced via
  branch protection.
- **Automated checks.** [PLACEHOLDER: tests/CI checks] must pass before a change can merge.
- **No direct production changes.** Production deployments occur only through
  [PLACEHOLDER: the CI/CD pipeline]; direct console or server access to modify production is
  restricted to [PLACEHOLDER: documented emergency exceptions, approved by role X].
- **Infrastructure changes.** [PLACEHOLDER: e.g. "Managed via Terraform, reviewed through the
  same PR process as application code."]
- **Rollback.** [PLACEHOLDER: describe how a bad change is rolled back].

## 4. Emergency changes

Exceptions to standard review (e.g. an urgent production fix) require [PLACEHOLDER: post-hoc
approval within N hours, documented rationale].

## 5. Review history

| Version | Date | Reviewed by | Changes |
|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | Initial version |
