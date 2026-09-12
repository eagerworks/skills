# HIPAA — Technical Safeguards (§164.312)

> Walks §164.312 end to end and maps each specification to concrete Rails + Node/TypeScript
> implementations. "Required" specs need direct evidence; "Addressable" specs need either
> implementation or a documented equivalent alternative — see `SKILL.md` Gotcha 1.

## Table of Contents

1. [Access control — §164.312(a)](#access-control-164312a)
2. [Audit controls — §164.312(b)](#audit-controls-164312b)
3. [Integrity — §164.312(c)](#integrity-164312c)
4. [Person or entity authentication — §164.312(d)](#person-or-entity-authentication-164312d)
5. [Transmission security — §164.312(e)](#transmission-security-164312e)

---

## Access control — §164.312(a)

**Unique user identification (Required).** Every person accessing PHI has their own login — no
shared service accounts used by multiple humans, no generic `admin`/`support` credential.

```ruby
# ❌ wrong — a shared credential means audit logs can't attribute access to a person
User.find_by(email: "support@yourcompany.com") # one login, many staff share it

# ✅ correct — every staff member has an individually attributable account
current_user.id # unique per person, traceable in every audit log entry
```

**Emergency access procedure (Required).** A documented way to get to PHI during an emergency
when normal access is unavailable — e.g. a break-glass account, logged and alerted separately.
Confirm one exists and is itself access-controlled and logged; an ungoverned "just SSH in"
fallback is a gap, not a safeguard.

**Automatic logoff (Addressable).** Sessions touching PHI expire after inactivity.

```ts
// ✅ correct — session TTL enforced server-side, not just client-side idle detection
const session = await store.get(sessionId)
if (Date.now() - session.lastActivity > SESSION_TTL_MS) {
  await store.destroy(sessionId)
  throw new UnauthorizedError('Session expired')
}
```

**Encryption and decryption (Addressable).** PHI at rest is encrypted — see
`references/infrastructure.md` for cloud-provider verification commands and BAA-eligible service
lists. Application-level column encryption for the most sensitive fields is stronger than relying
solely on disk-level encryption:

```ruby
# ✅ correct — Rails 7+ built-in encryption for a sensitive PHI column
class Patient < ApplicationRecord
  encrypts :diagnosis_notes
end
```

Missing this with no documented equivalent (e.g. full-disk encryption confirmed active,
compensating network isolation) is a 🔴 — "addressable" is not "optional," see `SKILL.md` Gotcha 1.

## Audit controls — §164.312(b)

Full treatment in `references/audit-controls.md`. Summary: hardware, software, and procedural
mechanisms that record and examine activity in systems containing PHI — **including read access**,
not just writes. Confirm logs are tamper-resistant and retained per the 6-year documentation rule
(§164.316(b)).

## Integrity — §164.312(c)

**PHI is protected from improper alteration or destruction (Addressable).** Look for:

- Database-level constraints and application validation preventing malformed writes to PHI tables.
- A mechanism to detect unauthorized alteration — checksums, versioned records, or an
  append-only audit trail alongside mutable records so a change can be reconstructed.
- Soft-delete rather than hard-delete on PHI records where feasible, so accidental or malicious
  destruction is recoverable and visible.

```ruby
# ❌ wrong — hard delete destroys the only record of what existed
patient.destroy

# ✅ correct — soft delete preserves the record; a real purge is a separate, logged, audited step
patient.update!(deleted_at: Time.current)
```

## Person or entity authentication (Required) — §164.312(d)

Verify that a person or entity seeking access to PHI is who they claim. In practice: strong
password policy or SSO, and MFA for any account with PHI access.

```ts
// ✅ correct — MFA enforced (not merely offered) before granting a PHI-scoped session
if (!user.mfaEnabled || !verifyMfaToken(user, providedToken)) {
  throw new UnauthorizedError('MFA required for PHI access')
}
```

Check that authentication is enforced server-side on every PHI-touching endpoint, not only on the
login page — a common gap is an API route reachable with a valid session token but no
re-verification of MFA status on the specific PHI-scoped action.

## Transmission security — §164.312(e)

**Integrity controls (Addressable) and encryption (Addressable) in transit.** TLS for every
channel PHI crosses — API traffic, internal service-to-service calls, email, webhooks, background
job payloads sent to a queue.

```ts
// ❌ wrong — PHI-bearing webhook sent over plain HTTP
await fetch(`http://partner-system.internal/webhook`, { method: 'POST', body: JSON.stringify(patientPayload) })

// ✅ correct — TLS enforced, and the endpoint itself is verified, not just "https://"
await fetch(`https://partner-system.internal/webhook`, { method: 'POST', body: JSON.stringify(patientPayload) })
```

```bash
# Verify a service doesn't accept plaintext connections
grep -rniE 'http://' --include='*.rb' --include='*.ts' --include='*.js' . | grep -viE 'localhost|127\.0\.0\.1|example\.com'
```

Check message queues, background job payloads (Sidekiq/Bull job args are often logged or stored
unencrypted by default), and internal service mesh traffic — "it's inside our VPC" is not the
same as "it's encrypted in transit," and both are worth confirming independently.
