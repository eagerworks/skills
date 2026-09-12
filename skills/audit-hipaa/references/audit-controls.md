# HIPAA — Audit Controls (§164.312(b))

> §164.312(b) in depth: what must be logged, how long, and the specific trap of an audit log that
> becomes its own PHI exposure. This is the safeguard teams most often half-implement — they log
> writes and assume that's the whole requirement.

## Table of Contents

1. [What must be logged](#what-must-be-logged)
2. [Reads count, not just writes](#reads-count-not-just-writes)
3. [Retention — the 6-year rule](#retention--the-6-year-rule)
4. [Tamper resistance](#tamper-resistance)
5. [The audit log as its own exposure](#the-audit-log-as-its-own-exposure)
6. [Concrete implementations](#concrete-implementations)

---

## What must be logged

§164.312(b) requires "hardware, software, and/or procedural mechanisms that record and examine
activity in information systems that contain or use" ePHI. In practice, for every PHI-touching
system, capture at minimum:

- **Who** — the authenticated user or service identity (see §164.312(d), unique user ID)
- **What** — which record(s) and which PHI fields were involved
- **When** — a timestamp
- **What action** — read, create, update, delete, export, print
- **From where** — source IP or originating service, useful for anomaly detection

## Reads count, not just writes

This is the most commonly missed half of the requirement. A change log (`created_at`,
`updated_at`, a `PaperTrail`/audit-gem history of mutations) satisfies the "what changed" half but
says nothing about who *viewed* a record without changing it — which is the shape of the most
common real-world incident: an employee browsing a patient chart they had no treatment reason to
access.

```ruby
# ❌ wrong — only mutations are audited; a curious/malicious read leaves no trace
class Patient < ApplicationRecord
  has_paper_trail # tracks create/update/destroy only
end

# ✅ correct — read access explicitly logged alongside mutation history
class PatientsController < ApplicationController
  def show
    @patient = Patient.find(params[:id])
    AuditLog.create!(user: current_user, action: 'read', resource: @patient, occurred_at: Time.current)
  end
end
```

```bash
# Check whether an audit/history gem is present and what it actually tracks
grep -rniE '(paper_trail|audited|logidze)' Gemfile 2>/dev/null
grep -rn 'has_paper_trail\|audited' app/models 2>/dev/null
```

## Retention — the 6-year rule

§164.316(b)(2)(i) requires HIPAA-related documentation — audit logs among it — be retained **6
years from creation or from when it was last in effect**, whichever is later. Two failure modes:

1. **Log retention shorter than 6 years** on the storage/log-aggregation side (a common default is
   30–90 days). Check the actual retention config, not the assumption that "logs are kept."
2. **Retention shorter than an active investigation or dispute window** — 6 years is a floor, not
   a ceiling; extend it if litigation hold or an active OCR inquiry requires longer.

```bash
# AWS CloudWatch Logs — check retention isn't left at the "never expire" default that quietly
# becomes a cost problem, nor set shorter than 6 years for PHI-access logs
aws logs describe-log-groups --query 'logGroups[].[logGroupName,retentionInDays]' --output table
```

## Tamper resistance

An audit log a privileged user can silently edit or delete defeats its own purpose. Prefer
append-only storage (a write-once log stream, a separate datastore the application's normal
credentials can't `DELETE` from) over a plain table row a superuser-level DB credential can alter.

```ruby
# ❌ wrong — audit log is a normal ActiveRecord table any admin-level DB access can edit
class AuditLog < ApplicationRecord; end

# ✅ correct — audit events also shipped to an append-only external sink the app can't mutate
class AuditLog < ApplicationRecord
  after_create { ExternalAuditSink.append(self) } # e.g. write-once S3 bucket with Object Lock,
                                                     # or a dedicated log aggregator with no delete API
end
```

## The audit log as its own exposure

Recording *what* a user accessed is necessary — recording the PHI *content itself* in the audit
log is a new exposure, often with weaker access controls than the source table, since audit logs
are frequently readable by a broader engineering/ops audience for debugging purposes.

```ruby
# ❌ wrong — the audit trail duplicates the PHI it's supposed to be watching over
AuditLog.create!(user: current_user, action: 'read', details: patient.diagnosis_notes)

# ✅ correct — records that access happened, without copying the sensitive content
AuditLog.create!(user: current_user, action: 'read', resource_type: 'Patient', resource_id: patient.id)
```

## Concrete implementations

```ts
// Node/TS — a lightweight audit middleware pattern for a PHI-scoped route
async function auditedRead(req: Request, patientId: string) {
  const patient = await db.patient.findUnique({ where: { id: patientId } })
  await db.auditLog.create({
    data: {
      userId: req.user.id,
      action: 'read',
      resourceType: 'Patient',
      resourceId: patientId,
      occurredAt: new Date(),
      sourceIp: req.ip,
    },
  })
  return patient
}
```

When auditing a codebase, grep for the PHI-touching controllers/routes identified in
`references/phi-in-code.md` and confirm each has a paired audit-write on its read path, not just
its write path — a route-by-route check, not a repo-wide assumption from one good example.
