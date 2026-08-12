# Access Control Policy

| | |
|---|---|
| **Owner** | [PLACEHOLDER: role, e.g. Head of Engineering] |
| **Approver** | [PLACEHOLDER: role] |
| **Review cadence** | Annual |
| **Last reviewed** | [PLACEHOLDER: date] |
| **Version** | [PLACEHOLDER: 1.0] |

> **Auditor note:** This is the most heavily tested policy (CC6). Auditors sample real
> provisioning/deprovisioning records and access review exports against what this policy says —
> mismatches are the single most common finding. See `references/technical-controls.md`.

## 1. Purpose

Defines how access to systems, infrastructure, and data is granted, reviewed, and revoked.

## 2. Scope

Applies to all systems in scope: [PLACEHOLDER — cloud provider console(s), source control,
production databases, admin panels, ticketing, etc.].

## 3. Policy statements

- **Authentication.** [PLACEHOLDER: e.g. "SSO via [provider] is required for all in-scope
  systems; MFA is enforced for all accounts."]
- **Least privilege.** Access is granted by role, not by individual ad hoc request, and limited
  to what's needed for the person's function.
- **Provisioning.** New access requests are [PLACEHOLDER: describe the actual process — e.g.
  requested via ticket, approved by the resource owner, granted within N business days].
- **Deprovisioning.** Access is revoked [PLACEHOLDER: describe — e.g. "within 1 business day of
  termination, triggered by the offboarding checklist"]. This includes SSO, VPN, source control,
  and any system not covered by SSO.
- **Periodic review.** Access to [PLACEHOLDER: in-scope systems] is reviewed [PLACEHOLDER:
  quarterly] by the resource owner; results are recorded and retained as evidence.
- **Privileged access.** Standing admin/root access is limited to [PLACEHOLDER: named
  roles/individuals] and reviewed with the same cadence as standard access.

## 4. Enforcement

Deviations require documented, time-bound exception approval by [PLACEHOLDER: role].

## 5. Review history

| Version | Date | Reviewed by | Changes |
|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | Initial version |
