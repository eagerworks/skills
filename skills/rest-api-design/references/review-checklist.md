# REST API Design — Reviewing an Existing API

> Covers auditing an API that's already shipped. For designing a new one, start at `SKILL.md` →
> Survey the Existing API First.

## Table of Contents

1. [Enumerate the surface](#enumerate-the-surface)
2. [Run the checklist](#run-the-checklist)
3. [Triage by severity](#triage-by-severity)
4. [Reporting findings](#reporting-findings)

---

## Enumerate the surface

Before judging anything, get a complete list of what exists — a review based on a sample of endpoints
will miss inconsistencies that only show up across the full surface:

```bash
# From an OpenAPI spec, if one exists and is trustworthy
grep -E "^\s+(get|post|put|patch|delete):" openapi.yaml

# From route definitions directly (more reliable if the spec might be stale)
grep -rnE "router\.|@(Get|Post|Put|Patch|Delete)Mapping|app\.(get|post|put|patch|delete)\(|resources :" \
  --include="*.rb" --include="*.ts" --include="*.js" --include="*.py" --include="*.java" --include="*.go" .
```

List every path + method pair before starting the checklist below — it's the review's scope, and
gives you the base to spot cross-endpoint inconsistency (e.g. half the endpoints paginate by cursor,
half by offset).

## Run the checklist

For each endpoint (or representative sample if the surface is very large — say so in the report if you
sampled rather than covered everything):

- [ ] Path uses nouns, not verbs (`resource-modeling.md`)
- [ ] Method matches semantics: safe/idempotent guarantees actually hold (`http-semantics.md`)
- [ ] Status codes are specific and correct (`400` vs `422`, `401` vs `403`, etc.)
- [ ] `201` responses include `Location`
- [ ] Error responses don't leak internals (stack traces, SQL, file paths)
- [ ] Error shape is consistent with the rest of the API (one envelope, not several)
- [ ] Collection endpoints have a page-size default *and* max
- [ ] Collection endpoints use pagination that's safe under concurrent writes (or explicitly accept the trade-off)
- [ ] IDs exposed externally are non-sequential, or protected by strict per-object authorization
- [ ] Every `/{resource}/{id}` handler checks object-level ownership, not just authentication
- [ ] No credentials/tokens in URLs or query strings
- [ ] Breaking changes (if any in scope) are behind a version bump, not shipped silently
- [ ] OpenAPI spec (if one exists) matches actual behavior for this endpoint

## Triage by severity

Report findings in this order — don't let a naming nitpick bury a security bug:

1. **Security / correctness** — IDOR, mass assignment, credential leakage, an idempotent method that
   isn't, a status code that actively misleads clients (`200` wrapping an error).
2. **Breaking-change risk** — an in-flight or recent change that breaks existing clients without a
   version bump.
3. **Consistency** — one endpoint pluralizes and another doesn't, mixed key casing, inconsistent error
   shapes across endpoints.
4. **Polish** — missing examples in the spec, a `422` that could be more specific, a field name that's
   slightly unclear.

## Reporting findings

State the concrete failure, not just the rule violated — "X breaks under Y" is more actionable than
"X isn't best practice":

```
❌ vague
"GET /invoices/{id} doesn't follow REST best practices for authorization."

✅ concrete
"GET /invoices/{id} checks `invoices:read` scope but not invoice ownership — any authenticated
user can read any other account's invoice by guessing/incrementing the ID (IDOR). File: 
routes/invoices.js:42."
```

An audit of a shipped API is not a mandate to rewrite it wholesale — group findings by severity (above),
and be explicit that consistency/polish findings are optional cleanup while security/correctness
findings need a fix. Match the reporting shape the codebase's other reviews use if one is established;
otherwise `assets/api-review-checklist.md` is a ready-to-copy version of the checklist above.
