# PR Review — Rubric

The authoritative checklist and severity ladder. Read this in full before writing a report —
`SKILL.md` only summarizes it. Examples below are drawn from both Rails and Node/TypeScript so
either studio recognizes its own code; apply the same lens regardless of stack.

## Lens 1 — Correctness

Logic errors, edge cases, off-by-ones, unhandled states or inputs, races between concurrent
operations, incorrect error handling (swallowed errors, wrong status codes, wrong fallback
behavior).

```ruby
# ❌ wrong — off-by-one, misses the last page
def paginate(items, page:, per_page: 20)
  items[(page - 1) * per_page...page * per_page - 1]
end
```

```ts
// ❌ wrong — error swallowed, caller can't tell the write failed
async function saveDraft(draft: Draft) {
  try {
    await db.draft.update({ where: { id: draft.id }, data: draft })
  } catch {
    console.log("save failed")
  }
}
```

## Lens 2 — Security & Data Integrity

**Multi-tenant / auth scoping.** Every query and mutation touching tenant- or user-scoped data
filters by the caller's identity, not just by an id supplied in the request. A missing scope
check is always `critical`.

```ruby
# ❌ wrong — any authenticated user can fetch any org's invoice by guessing the id
def show
  @invoice = Invoice.find(params[:id])
end

# ✅ correct — scoped through the caller's own organization
def show
  @invoice = current_user.organization.invoices.find(params[:id])
end
```

```ts
// ❌ wrong — orgId never enters the query
const project = await prisma.project.findUnique({ where: { id: params.id } })

// ✅ correct
const project = await prisma.project.findFirst({
  where: { id: params.id, orgId: session.orgId },
})
```

**Injection.** SQL/command/template injection from unsanitized input.

```ruby
# ❌ wrong — string-interpolated SQL
Invoice.where("customer_name LIKE '%#{params[:q]}%'")

# ✅ correct — bound parameter
Invoice.where("customer_name LIKE ?", "%#{params[:q]}%")
```

```ts
// ❌ wrong — raw query built with interpolation
await prisma.$queryRawUnsafe(`SELECT * FROM users WHERE email = '${email}'`)

// ✅ correct — tagged template, parameters are bound
await prisma.$queryRaw`SELECT * FROM users WHERE email = ${email}`
```

**Secrets.** No credential, token, or key written to logs, error messages, or a client-visible
response.

```ts
// ❌ wrong — leaks the API key into application logs on every failure
logger.error(`Stripe call failed with key ${stripeApiKey}: ${err.message}`)
```

**Migration safety.** A schema migration doesn't lock or lose data on a table that already has
rows in production; a destructive migration is paired with a backfill or an explicit rollback
path.

```ruby
# ❌ wrong — locks the table for the duration of the index build
add_index :orders, :customer_id

# ✅ correct — Rails/PG allow building without a full table lock
add_index :orders, :customer_id, algorithm: :concurrently
```

```ts
// ❌ wrong — NOT NULL with no default on a populated table fails outright
await queryRunner.query(`ALTER TABLE "orders" ADD COLUMN "status" varchar NOT NULL`)

// ✅ correct — backfill first, then tighten the constraint in a follow-up migration
await queryRunner.query(`ALTER TABLE "orders" ADD COLUMN "status" varchar DEFAULT 'pending'`)
```

## Lens 3 — Repo-Convention Conformance

Cross-check against this repo's own `AGENTS.md`/`CLAUDE.md`, and against any project-specific
pattern evident from the surrounding code that this change breaks without a stated reason —
existing service-object structure, error-handling style, naming, file organization.

**Code that follows a documented convention is never a finding**, even if a different style is
generally preferable. Only flag a deviation, not a preference.

## Lens 4 — Test Coverage

For every acceptance criterion in the issue or PR description (or, if none is given, for every
new behavior the diff introduces), find the test that proves it and confirm it actually
**asserts** the behavior — not just exercises the code path.

```ts
// ❌ does NOT count as covering "a user cannot read another org's project"
it("returns a project", async () => {
  const res = await request(app).get(`/projects/${project.id}`).set(authHeader(user))
  expect(res.status).toBe(200) // never asserts the scoping behavior
})

// ✅ actually proves the acceptance criterion
it("404s when the project belongs to a different org", async () => {
  const res = await request(app).get(`/projects/${otherOrgProject.id}`).set(authHeader(user))
  expect(res.status).toBe(404)
})
```

An acceptance criterion with no proving test is always `high`.

## Severity Ladder

- **critical / high** — correctness bugs, security or data-integrity gaps, or a change touching
  a lot of logic (state transitions, auth/scoping guards, migrations); an uncovered acceptance
  criterion.
- **minor** — local style/naming nits with no behavioral risk.
- **Not a finding** — anything that conflicts with a documented repo convention the code
  correctly follows, or a preference with no concrete failure path or cited convention behind
  it.

## Conservatism

Every finding needs either a concrete failure scenario (specific inputs or state that produce a
wrong result or crash) or a cited convention it violates. A preference with no behavioral or
documented-convention basis is not a finding — leave it out rather than padding the list. Zero
findings on a clean diff is the correct, expected outcome, not a sign the review was too
shallow.

## Repo-Specific Focus (extraFocus)

If `.eagerworks/pr-review.json` supplies `review.extraFocus` entries (see
`references/config.md`), treat each as an additional required check on top of the four lenses
above — read them as repo-specific known risk areas, e.g. "every quiz-scoped query must filter
by `orgId`, not just `quizId`".
