# Design QA Test Cases — Techniques

The authoritative checklist. `SKILL.md` only summarizes this table — read this in full before designing a plan. Every stack example appears in both Rails and Node/TypeScript so either studio recognizes its own code; the same reasoning applies unchanged to any other stack.

## Lens 1 — Happy Paths Per Acceptance Criterion

One case per AC, phrased as the AC's own success condition — not a paraphrase of the ticket title. If a feature has three ACs and no edge behavior worth calling out, three happy-path cases is a complete Lens 1, not an unfinished one.

```markdown
AC: "A user with a valid coupon code sees the discount applied at checkout."

Case — Valid coupon applies the discount
Verifies: AC-1
Steps: Add an item to the cart → enter a valid, unused, unexpired coupon → view checkout.
Expected: The discount is applied to the subtotal and shown as a separate line.
```

## Lens 2 — Equivalence Partitioning & Boundary Values

Never guess a boundary — read the validation. A ✅ boundary case cites the exact rule; a ❌ one guesses a round number that isn't actually where the code branches.

```ruby
# ❌ wrong — case says "cart total must be at least $10", invented
# ✅ correct — read the actual validation first
validates :subtotal, numericality: { greater_than_or_equal_to: 10 }
# → boundary cases at $9.99 (reject) and $10.00 (accept), not $0 or $100
```

```typescript
// ✅ correct — Zod schema is the source of the boundary, not intuition
const CouponCode = z.string().min(4).max(20).trim();
// → cases: 3 chars (reject), 4 chars (accept), 20 chars (accept), 21 chars (reject),
//   leading/trailing whitespace (trimmed then validated), empty string, unicode input
```

Standard partitions to check for every bounded input: empty, whitespace-only, exactly at each limit, one past each limit, unicode/emoji, and (for numeric fields) zero and negative values even when the domain "obviously" excludes them — the validation, not the domain story, decides whether that's tested as a rejection or an assumption worth flagging.

## Lens 3 — Negative & Error Paths

For every external dependency and every user input, add the case where it fails.

```ruby
# ✅ correct — the code has a rescue, so a case must exercise it
def apply_coupon(code)
  coupon = Coupon.active.find_by!(code: code)
  # ...
rescue ActiveRecord::RecordNotFound
  errors.add(:coupon, "is invalid or expired")
end
# → case: an expired or unknown code shows the exact error, not a 500
```

```typescript
// ✅ correct — a timeout branch in the code means a timeout case in the plan
try {
  const result = await paymentGateway.charge(order);
} catch (err) {
  if (err instanceof TimeoutError) return retryOrFail(order);
}
// → case: gateway timeout triggers the documented retry/fail behavior, not a hang
```

Also cover: malformed/missing required fields, duplicate submission (double-click), and a dependency returning an unexpected shape (a 200 with an empty body, a webhook with a missing field).

## Lens 4 — Permissions & Roles Matrix

One case per (role × action) pair the code actually distinguishes — not the full combinatorial cross product of every role and every action, which pads the plan with untestable duplicates.

```ruby
# ✅ correct — the policy is the source of the matrix, not a guess at "probably admin-only"
class CouponPolicy < ApplicationPolicy
  def redeem?
    user.member_of?(record.organization)
  end
  def void?
    user.admin_of?(record.organization)
  end
end
# → redeem?: member (allow), non-member (deny), Lens 3 case for a cross-org user
# → void?: admin (allow), member (deny) — this is the pair the policy actually distinguishes
```

```typescript
// ✅ correct — middleware order matters; test what it actually enforces
router.delete("/coupons/:id", requireRole("admin"), requireOrg, deleteCoupon);
// → admin in the coupon's own org (allow), admin in a different org (deny — requireOrg
//   still applies after requireRole), member (deny before reaching requireOrg)
```

## Lens 5 — State Transitions

Every enum or state machine the feature touches gets a transition table: legal transitions as happy-path cases, illegal ones as negative cases.

```ruby
# ✅ correct
enum status: { draft: 0, active: 1, expired: 2, redeemed: 3 }
# → legal: draft → active, active → redeemed, active → expired (by cron)
# → illegal: redeemed → active (a used coupon can't be reactivated by re-submitting),
#   expired → redeemed (an expired coupon can't be redeemed even mid-request)
```

A state with no reachable transition into it (a status the code sets but nothing ever reads) is worth flagging as an ambiguity, not silently skipped.

## Lens 6 — Data & Lifecycle

CRUD interactions beyond the single record: cascades, soft deletes, uniqueness under concurrency, and idempotency of any endpoint that can be retried.

```ruby
# ✅ correct — a unique index implies a race case, not just a validation case
add_index :coupon_redemptions, [:coupon_id, :user_id], unique: true
# → case: two concurrent requests to redeem the same coupon as the same user — exactly
#   one succeeds, the other gets a clean error, not a 500 or a duplicate redemption
```

```typescript
// ✅ correct — an endpoint with a client-supplied idempotency key needs a repeat-call case
app.post("/orders", idempotent("Idempotency-Key"), createOrder);
// → case: the same request replayed with the same key returns the original result,
//   not a second order
```

Also cover: deleting a parent record with dependent children (cascade vs. orphan vs. restrict), and a soft-deleted record's visibility in every place it's queried.

## Lens 7 — Integration & Async

Background jobs, webhooks, and third-party calls fail asynchronously from the user's perspective — cases must cover both the immediate response and the eventual outcome.

```ruby
# ✅ correct
class SendReceiptJob < ApplicationJob
  retry_on Net::ReadTimeout, wait: :exponentially_longer, attempts: 3
end
# → case: the email provider times out once, the job retries and eventually succeeds
# → case: it fails all 3 attempts — what does the user see, and is it alerted anywhere?
```

Also cover: a webhook received out of order or twice (idempotency again, from the other side), and eventual-consistency windows — an action that immediately shows one state but whose downstream effect (search index, cache, read replica) lags.

## Lens 8 — Cross-Cutting UX

Only testable with a design source (Figma, screenshot, or an existing equivalent screen in the app) or an explicit written spec — see "Ambiguities" below when neither is available.

- **Responsive** — the breakpoints the design actually defines, not an arbitrary set.
- **Accessibility** — keyboard-only completion of the flow, screen-reader labels on new interactive elements, color contrast on new states (error, disabled, success).
- **i18n & locale** — a string that changes length in another locale doesn't break layout; date/number formatting follows locale, not the developer's.
- **Timezone** — any date shown or compared crosses a timezone boundary correctly (a "coupon expires today" case evaluated near midnight in the user's zone, not the server's).
- **Empty / loading / error states** — every new screen or component has all three specified, not just the happy-path render.

## Traceability

Every case carries a `Verifies:` field — the AC it satisfies, or `code: <signal>` when it was derived from a validation/policy/enum with no matching AC. This is **not** a full AC × case matrix; it's a gap list:

```markdown
### Acceptance criteria with no case

- **AC-3** "Expired coupons are rejected" — not testable as written; see Ambiguities below.
```

An AC with at least one case needs no separate entry — the case list itself is the proof. See `docs/decision-records/2026-09-07--test-cases-carry-no-synthetic-ids.md` for why this is a gap list and not a matrix keyed by synthetic IDs.

## Ambiguities

An AC — or a cross-cutting UX area with no design — that can't be turned into a concrete pass/fail case becomes a question for the PO, never an invented assumption:

```markdown
### Open questions for the PO

1. AC-3 ("Expired coupons are rejected") doesn't define expiry — calendar day in the
   user's timezone, or a UTC instant? The test at the boundary depends on which.
```

This is this skill's equivalent of `pr-review`'s conservatism rule: a confidently wrong assumption is worse than a stated gap. Never silently pick the reading that's easiest to test.

## Conservatism

Every case needs a concrete trigger: an AC, a validation at `file:line`, a policy, a state enum, or a `testCases.extraFocus` entry from config. Zero cases beyond the happy paths for a feature with no bounded inputs, no roles, and no state machine is the correct, expected outcome — never pad the P1/P2 sections with generic checklist items ("test on mobile", "check loading spinner") that don't trace to anything this feature actually touches.
