# GDPR — Personal Data in Code

> The highest-yield audit surface: personal data doesn't usually leak through the database
> schema, it leaks through the plumbing around it — logs, error trackers, analytics, LLM calls,
> and test fixtures. Work through each of these before anything else; they produce the most
> findings per hour of review.

## Table of Contents

1. [Application logs](#application-logs)
2. [Exception trackers](#exception-trackers)
3. [Analytics and telemetry SDKs](#analytics-and-telemetry-sdks)
4. [LLM prompts and embeddings](#llm-prompts-and-embeddings)
5. [URLs, query strings, and client-visible errors](#urls-query-strings-and-client-visible-errors)
6. [Email and SMS](#email-and-sms)
7. [Seeds, fixtures, and test data](#seeds-fixtures-and-test-data)

---

## Application logs

`.inspect`, `.to_s`, `JSON.stringify`, or an ORM object dropped straight into a log call prints
every attribute the model has, whether or not the log statement's author meant to include it.
Logs also typically have longer, less-controlled retention than the primary database — a
personal-data-bearing log line can outlive an erasure request by months if log retention isn't
itself part of the erasure flow.

```
# ❌ wrong — full record, including email/address/phone, into a shared log stream
logger.info("Order placed: #{order.inspect}")
# ✅ correct — identifiers only, no attribute payload
logger.info("Order placed: order_id=#{order.id} user_id=#{order.user_id}")
```

Check framework-level parameter filtering too (Rails `config.filter_parameters`, a Node
middleware's redaction list) — a filter that only lists `password` and misses `email`, `address`,
`ssn`, or a custom PII field is a silent gap the same shape as no filter at all.

## Exception trackers

Sentry, Bugsnag, Rollbar, and similar tools capture request context by default — headers, query
params, form bodies, sometimes full request/response payloads — which routinely includes personal
data the developer never explicitly logged. A default-configured error tracker on a signup or
checkout endpoint captures names, emails, and addresses in every exception it records, indefinitely,
in a system the audit's own access controls may not cover at all.

```
# ❌ wrong — default config, no scrubbing
Sentry.init(dsn: ENV["SENTRY_DSN"])
# ✅ correct — explicit scrub of known personal-data fields before send
Sentry.init(dsn: ENV["SENTRY_DSN"]) do |config|
  config.before_send = lambda do |event, hint|
    event.request&.data&.except!("email", "phone", "address", "ssn")
    event
  end
end
```

## Analytics and telemetry SDKs

`.identify()` and `.track()` calls in product analytics (Segment, Amplitude, Mixpanel, PostHog)
routinely pass a full user-traits object, which can include special-category fields (health,
religion, orientation) if the app's own data model has them, without anyone treating the
analytics pipeline as a place special-category data needs a separate check. Session-replay tools
(FullStory, Hotjar, LogRocket) capture DOM content by default, including form field values, unless
masking rules are explicitly configured for personal-data-bearing fields.

A distinct trap in the same code: an embedded ad pixel, a "Like"/social button, or a conversion
API script (`<script src="https://connect.facebook.net/...">` and similar) doesn't just send data
to a processor — for the collection-and-transmission phase specifically, the org and the platform
can both be determining the purposes and means, which makes them **joint controllers** under
Art 26, not a controller/processor pair. That's a different obligation (a transparent arrangement
allocating responsibilities, Art 26(1)/(2)) than an Art 28 contract, and a report that files every
third-party script as "needs a DPA" mischaracterizes this specific, common pattern.

## LLM prompts and embeddings

A prompt built by interpolating a user record into a string, or an embedding generated from raw
personal data and stored in a vector database, is personal data leaving the org's direct control
the same way any other outbound API call is — see `SKILL.md` Gotcha 6 for the processor-contract
requirement this triggers. Two code-visible failure modes beyond the contract question: a system
prompt or few-shot example that hardcodes a real customer's data as a template, and an embedding
store with no deletion path — vector databases are frequently omitted from erasure flows because
they don't look like a "database of personal data" to whoever built the erasure endpoint.

```
# ❌ wrong — raw PII interpolated into an outbound prompt, no scrubbing
prompt = "Summarize this customer's issue: #{ticket.customer_name}, #{ticket.customer_email}: #{ticket.body}"
# ✅ correct — identifiers stripped, ticket body summarized on its own merits
prompt = "Summarize this support issue: #{ticket.body}"
```

## URLs, query strings, and client-visible errors

Personal data in a URL (an email in a query param, a reset token that embeds an identifiable
value) ends up in server access logs, browser history, referrer headers sent to third-party
scripts on the next page, and any CDN or proxy logging layer in between — all surfaces the
application's own logging config doesn't control. Client-visible error messages that echo back
submitted data ("No account found for jane.doe@example.com") leak personal data to whoever is
looking at the screen, which matters for shared devices and support screen-shares.

## Email and SMS

Transactional email/SMS providers (SendGrid, Twilio, Postmark) receive full message content by
default, and many retain it for debugging/analytics beyond the send itself — check whether the
account-level retention setting has been reviewed, and whether email templates interpolate more
personal data into the body than the message actually needs (a password-reset email doesn't need
the user's full address in the footer just because the template happens to have access to it).

## Seeds, fixtures, and test data

`db/seeds.rb`, factory files, and fixture data drawn from a production database export are a
common, overlooked personal-data store — often checked into version control, often containing
real names and emails from an early data pull that nobody re-anonymized before committing.

```
# ❌ wrong — a real customer record copy-pasted into a fixture, now in git history permanently
user = User.create!(email: "jane.doe@realcompany.com", name: "Jane Doe", ssn: "078-05-1120")
# ✅ correct — synthetic data, generated per-run
user = User.create!(email: Faker::Internet.email, name: Faker::Name.name)
```

If real personal data is found in git history (not just the current tree), flag it as a 🔴 and
note that removing it from the working tree does not remove it from history — that needs a
separate remediation the audit should name but not perform (a `git filter-branch`/BFG-style
rewrite plus a forced push is a destructive action outside a read-only audit's scope).
