# SOC 2 — Technical Controls Reference

> Maps the Common Criteria to concrete engineering work. This is the "go build this" file — use
> it once scope is known, to turn CC5–CC8 into an actual technical control list. Examples use
> AWS/GCP-flavored services as illustrations; substitute the organization's actual provider.

## Table of Contents

1. [Identity & access (CC6)](#identity--access-cc6)
2. [Logging, monitoring & alerting (CC7)](#logging-monitoring--alerting-cc7)
3. [Encryption (CC6)](#encryption-cc6)
4. [Vulnerability management (CC7)](#vulnerability-management-cc7)
5. [Backups & recovery testing (CC9)](#backups--recovery-testing-cc9)
6. [Change management (CC8)](#change-management-cc8)
7. [Endpoint management (CC6)](#endpoint-management-cc6)
8. [Network controls (CC6, CC7)](#network-controls-cc6-cc7)

---

## Identity & access (CC6)

- **SSO** for every system touching production or customer data — a cloud console, source
  control, ticketing, and any internal admin panel. Single point of provisioning/deprovisioning
  beats managing accounts per-tool.
- **MFA enforced**, not just available — auditors will ask for a report showing enforcement, not
  a policy saying it's encouraged.
- **Least privilege role design** — named roles (e.g. `read-only-analyst`, `deploy-engineer`)
  rather than ad hoc per-person permissions, so access reviews can reason about roles instead of
  auditing every individual grant from scratch.
- **Deprovisioning on offboarding** — the single most commonly failed control. Tie it to the HR
  offboarding process directly (e.g. an HRIS webhook or a required step in the offboarding
  checklist that revokes SSO, which cascades to connected systems) rather than relying on someone
  remembering.
- **Quarterly access reviews** — export current access per system, have the resource owner
  confirm each grant is still appropriate, keep the signed-off export as evidence. See
  `evidence-and-monitoring.md` for the recurring calendar.

```bash
# Example: export current IAM users and their group membership for a quarterly access review
aws iam list-users --query 'Users[].[UserName,CreateDate]' --output table
aws iam list-groups-for-user --user-name jane.doe

# Example: confirm MFA is enforced (not just enabled) for every IAM user
aws iam list-virtual-mfa-devices --query 'VIRTUALMFADevices[].SerialNumber'
```

```yaml
# ✅ correct — access granted by named role, reviewable at a glance
role: deploy-engineer
permissions: [deploy:production, logs:read]
# ❌ wrong — one-off grant with no role to reason about during a review
user: jane.doe
permissions: [ec2:*, s3:*, iam:*]   # accumulated ad hoc over time, never revisited
```

## Logging, monitoring & alerting (CC7)

- **Centralize logs** somewhere queryable and retained — cloud-native (AWS CloudTrail + a log
  sink, GCP Cloud Audit Logs, Azure Monitor) is usually enough without a dedicated SIEM at
  startup scale.
- **Alert on the events that matter**: failed login spikes, privilege escalation, security-group
  or IAM policy changes, unusual data export volume. A handful of high-signal alerts beats a
  noisy firehose nobody reads.
- **Retention long enough to cover the observation window** — if logs roll off after 30 days but
  the Type II window is 6 months, there's no way to evidence months 1–5 later. Set retention
  before the window starts, not after an auditor asks.

```bash
# Example: confirm CloudTrail is enabled and check its retention/log-file destination
aws cloudtrail describe-trails --query 'trailList[].[Name,S3BucketName,IsMultiRegionTrail]'

# Example: a high-signal alert — notify on any IAM policy change (privilege escalation risk)
aws cloudwatch put-metric-alarm \
  --alarm-name iam-policy-change \
  --metric-name IAMPolicyEventCount \
  --namespace CloudTrailMetrics \
  --statistic Sum --period 300 --threshold 1 --comparison-operator GreaterThanOrEqualToThreshold
```

## Encryption (CC6)

- **In transit** — TLS everywhere, including internal service-to-service traffic where
  practical; no plaintext HTTP for anything touching customer data.
- **At rest** — provider-managed encryption (e.g. AWS RDS/S3 default encryption, GCP default
  encryption at rest) is a legitimate control, but verify it's actually turned on rather than
  assuming a managed service enables it by default for every resource type.
- **Key management** — document who can access encryption keys and how key rotation works, even
  if using a fully managed KMS; "the cloud provider manages it" is a valid answer only when it's
  actually true for the specific service in question.

```bash
# Example: verify at-rest encryption on an RDS instance and an S3 bucket — don't assume
aws rds describe-db-instances --db-instance-identifier your-db \
  --query 'DBInstances[0].StorageEncrypted'
aws s3api get-bucket-encryption --bucket your-app-bucket
```

## Vulnerability management (CC7)

- **Recurring scanning** — dependency scanning in CI (e.g. Dependabot, Snyk, `npm audit`/`pip-audit`
  equivalents) plus periodic infrastructure/container scanning.
- **A patch SLA** — a documented, followed timeframe for remediating findings by severity (e.g.
  critical within 7 days, high within 30) — the SLA existing and being met is the evidence, not
  just the scan results themselves.
- **Annual third-party penetration test** — near-universal Type II requirement; schedule early,
  third-party testers commonly have multi-week lead times.

## Backups & recovery testing (CC9)

- **Automated backups** for anything holding customer data, with a defined retention period.
- **Restore testing** — a backup that's never been restored isn't evidence of anything; a
  periodic (at minimum annual) test restore, with the result recorded, is what auditors look for.
- **Documented RPO/RTO** as part of the BCDR plan (see `assets/policies/business-continuity-and-dr-plan.md`),
  even if informal — a number the team has actually thought about beats no number at all.

## Change management (CC8)

- **Required PR review before merge** — branch protection rules enforcing at least one approval,
  applied to the branch that deploys to production.
- **CI gating** — tests and any required checks must pass before merge/deploy; document what's
  gated and what isn't.
- **No direct-to-production pushes** — if anyone can bypass the pipeline, that's a gap regardless
  of how rarely it happens; either remove the capability or document and restrict it tightly with
  a clear exception process.
- **Infrastructure as code with review**, where practical — console changes to production
  infrastructure leave no reviewable trail; IaC (Terraform, CloudFormation, Pulumi) run through
  the same PR process closes that gap.

```bash
# Example: verify branch protection actually requires review before merge (don't just trust the docs)
gh api repos/your-org/your-app/branches/main/protection \
  --jq '.required_pull_request_reviews.required_approving_review_count'
```
```yaml
# ✅ correct — required checks + required review, enforced by the platform
branch_protection:
  main:
    required_approving_review_count: 1
    required_status_checks: [ci/test, ci/lint]
    enforce_admins: true
# ❌ wrong — admins can bypass, so "required" review isn't actually required for everyone
branch_protection:
  main:
    required_approving_review_count: 1
    enforce_admins: false
```

## Endpoint management (CC6)

- **Disk encryption, screen lock, and OS patching** enforced on any device with access to
  production systems or customer data — mobile device management (MDM) tooling if the team is
  large enough to need it; a documented baseline and manual verification if not.
- **Company-managed devices** for anyone with production access, rather than unmanaged personal
  devices, where feasible.

## Network controls (CC6, CC7)

- **Least-exposed network posture** — production databases and internal services bound to
  private networks/security groups, not open to the public internet; bastion hosts or VPN for
  any administrative access that needs to cross the boundary.
- **Segmentation** between environments (production isolated from staging/dev) so a compromise in
  one doesn't trivially reach the other.
