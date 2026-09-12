# HIPAA — Infrastructure

> BAA-eligible service lists per cloud, encryption verification commands, and network isolation
> checks. Vendor lists and specific service names below were current as of **August 2026** —
> confirm against the provider's own current HIPAA-eligible services page before citing one as
> fact; providers add and occasionally remove services from these lists.

## Table of Contents

1. [BAA-eligible services are opt-in, not automatic](#baa-eligible-services-are-opt-in-not-automatic)
2. [AWS](#aws)
3. [Google Cloud](#google-cloud)
4. [Azure](#azure)
5. [Encryption verification](#encryption-verification)
6. [Network isolation](#network-isolation)
7. [Backups and the 6-year retention interaction](#backups-and-the-6-year-retention-interaction)
8. [Common third-party vendors](#common-third-party-vendors)

---

## BAA-eligible services are opt-in, not automatic

A cloud provider signing a master BAA with an organization does not automatically cover every
service the org uses — providers publish a specific list of services eligible for use with PHI
under their BAA, and using a service *outside* that list with PHI is a violation even if the
organization has a signed BAA for the account overall. Confirm both: (1) a BAA is signed with the
provider, and (2) each specific service actually storing/processing PHI appears on that provider's
current eligible-services list.

## AWS

Representative HIPAA-eligible services (verify current list — AWS updates this periodically):
RDS, S3, EC2, Lambda, DynamoDB, EBS, EFS, ElastiCache, Redshift, SQS, SNS, CloudTrail,
Elasticsearch/OpenSearch Service, Bedrock (for AI/ML workloads — relevant if an LLM call routes
through it, see `references/phi-in-code.md`).

```bash
# Confirm a BAA is in place (organization-level; not verifiable purely from code —
# but this at least confirms Organizations/Artifact access, a precondition)
aws organizations describe-organization 2>/dev/null
```

Notably **not** eligible for PHI under AWS's BAA unless explicitly confirmed otherwise: consumer
services outside the published list, and any service used in a region or configuration the BAA
doesn't cover. When in doubt, treat "is this service BAA-eligible" as ⚪ needing human
confirmation against AWS's current list — don't assert a stale or memorized list as fact.

## Google Cloud

Representative HIPAA-eligible services: Compute Engine, Cloud SQL, Cloud Storage, BigQuery,
Cloud Functions, Pub/Sub, Cloud Logging, Vertex AI (for AI/ML workloads).

```bash
aws --version >/dev/null 2>&1 || true  # placeholder note: no cross-cloud CLI check exists here —
                                         # confirm BAA status via the Google Workspace/Cloud admin console
```

Google requires the organization to review and accept the Cloud Healthcare API / BAA terms
explicitly per project — a BAA at the organization level doesn't automatically extend to every
GCP project without this being confirmed.

## Azure

Representative HIPAA-eligible services: Virtual Machines, Azure SQL Database, Blob Storage, Azure
Functions, Service Bus, Azure Monitor, Azure OpenAI Service (for AI/ML workloads).

Microsoft publishes HIPAA implementation guidance and a BAA is included in the Microsoft Online
Services Terms for in-scope services — confirm the specific service used appears in the current
in-scope list rather than assuming coverage.

## Encryption verification

Don't assume a managed service enables encryption by default for every resource type — verify.

```bash
# AWS — RDS at-rest encryption
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,StorageEncrypted]' --output table

# AWS — S3 bucket default encryption
aws s3api get-bucket-encryption --bucket your-phi-bucket

# AWS — EBS volume encryption
aws ec2 describe-volumes --query 'Volumes[].[VolumeId,Encrypted]' --output table

# GCP — Cloud SQL encryption is on by default for new instances, but confirm CMEK usage if required
gcloud sql instances describe your-instance --format='value(diskEncryptionConfiguration)'

# Azure — confirm Storage Service Encryption is enabled (on by default, verify not disabled)
az storage account show --name yourstorageaccount --query 'encryption'
```

## Network isolation

PHI-holding databases and internal services should not be reachable from the public internet.

```bash
# AWS — check an RDS instance isn't publicly accessible
aws rds describe-db-instances --query 'DBInstances[].[DBInstanceIdentifier,PubliclyAccessible]' --output table

# AWS — check a security group doesn't have an open ingress rule to 0.0.0.0/0 on a DB port
aws ec2 describe-security-groups --query 'SecurityGroups[].IpPermissions[?FromPort==`5432`]'
```

```yaml
# ❌ wrong — database security group open to the world
IngressRule:
  Protocol: tcp
  FromPort: 5432
  ToPort: 5432
  CidrIp: 0.0.0.0/0

# ✅ correct — restricted to the application's own security group / private subnet
IngressRule:
  Protocol: tcp
  FromPort: 5432
  ToPort: 5432
  SourceSecurityGroupId: sg-app-servers
```

## Backups and the 6-year retention interaction

Confirm automated backups exist for every PHI datastore, with encryption applied to the backup
itself (not just the live database) and access controls matching the source data. Backup
retention interacts with §164.316(b)'s 6-year documentation rule for audit-relevant data — a
backup rotation policy that purges after 30 days is fine for disaster-recovery purposes but is a
separate question from audit-log retention, which needs its own 6-year-minimum policy (see
`references/audit-controls.md`).

## Common third-party vendors

Beyond the three major clouds, check BAA status for any vendor PHI can reach: transactional email
(SendGrid, Postmark), SMS (Twilio), observability (Datadog, New Relic), error tracking (Sentry,
Bugsnag), and LLM/AI providers. Most enterprise-tier plans of these vendors offer a BAA on
request, but it is **not standard on free/lower tiers** and is never automatic — confirm one is
signed rather than assuming enterprise-grade security posture implies it.
