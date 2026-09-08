# Design QA Test Cases — Code Scan

What justifies installing this skill in the project's own repo instead of asking a generic
chat for test cases: it reads the actual code the feature touches before writing a single
case. This is what to look for and what each signal yields.

## Stack Detection

Reuse the same manifest signals `loop-engineering-audit` classifies a repo by — don't
invent a second taxonomy:

| Signal | Stack | Where the relevant code lives |
|---|---|---|
| `Gemfile`, `config/application.rb` | Rails | `app/models/`, `app/policies/`, `app/services/`, `spec/` |
| `package.json` (+ `tsconfig.json`) | Node / TypeScript | `src/`, schema files (Zod/Yup), middleware, `*.test.ts` / `*.spec.ts` |
| `pyproject.toml`, `requirements*.txt` | Python | models/serializers, `tests/` |
| `go.mod` | Go | struct validation, `_test.go` |

A monorepo can hold several stacks and several test frameworks — scan the package(s) the
feature's diff or ticket actually touches, not the whole tree.

## Signal → Yield

| Signal in the repo | Yields |
|---|---|
| Model/schema validations — ActiveRecord `validates`, Zod/Yup schemas, DB `CHECK`/`NOT NULL`/unique constraints | Lens 2 boundary values, with the exact limit cited at `file:line` |
| Policies, guards, `before_action`, middleware, RLS policies | Lens 4's role × action matrix — only the pairs the code actually distinguishes |
| Enums, state machines (`aasm`, `statesman`, a `status` column with a comment listing values, a TypeScript union type) | Lens 5's legal/illegal transition table |
| Feature flags (LaunchDarkly, Flipper, a `feature_flags` table, an env-gated `if`) | Each affected case duplicated per flag state — on and off are two different features to QA |
| Background jobs, queues, webhooks, scheduled tasks | Lens 7's async and outage cases |
| External API clients (payment gateways, email/SMS providers, third-party SDKs) | Lens 7's timeout/outage/malformed-response cases |
| i18n locale files (`config/locales/*.yml`, `messages/*.json`) | Lens 8's locale coverage — only the locales the repo actually ships |
| Migrations in the diff | A data-migration/backfill case: does existing data satisfy the new constraint? |
| **Existing test files** | **Coverage cross-reference — see below** |

## Coverage Cross-Reference

The differentiator between this skill and a plain "list some test cases" prompt: before
emitting a case, check whether an existing automated test already asserts it.

1. Locate the test files most likely to cover the feature — same directory structure as
   the changed source (`app/models/coupon.rb` → `spec/models/coupon_spec.rb`;
   `src/checkout/apply-coupon.ts` → `src/checkout/apply-coupon.test.ts`), plus any e2e
   spec that exercises the same user flow.
2. For each candidate case, check whether an existing `it`/`test`/`describe` block already
   asserts that exact behavior — not just that the file *touches* the same code.
3. Mark the case accordingly in the report:
   - `✅ covered — spec/models/coupon_spec.rb:42` — an existing test already asserts this;
     still listed (so the plan stays a complete picture of what protects the feature), but
     QA doesn't need to manually re-verify it every release.
   - `New` — no existing test covers it; this is where QA effort (manual or newly
     automated) actually needs to go.

This is toggleable via `testCases.automation.crossReferenceExistingTests` (default `true`
— see `references/config.md`). Turning it off means every case is reported `New`, and the
report says so in one line rather than silently.

**A skipped/pending/`.only`'d test doesn't count as covering anything.** `it.skip`, `xit`,
`describe.skip`, `test.todo`, RSpec `pending`, or a sibling test file with `.only` left in
it means the assertion isn't actually running — cite it as `New` with a note, not `✅ covered`.
