# HIPAA — PHI in Code

> The highest-yield audit surface: PHI doesn't usually leak through the database, it leaks through
> the plumbing around it — logs, error trackers, analytics, LLM calls, and test fixtures. Work
> through each of these before anything else; they produce the most findings per hour of review.

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

The single most common PHI leak: a `logger` call that includes a full model object, params hash,
or interpolated PHI field.

```ruby
# ❌ wrong — full patient object (name, DOB, diagnosis) written to plaintext logs
logger.info("Updated record: #{patient.inspect}")

# ❌ wrong — PHI interpolated directly
logger.info("Sent reminder to #{patient.email} for appointment #{appointment.id}")

# ✅ correct — identifiers only, no PHI content
logger.info("Sent reminder for appointment_id=#{appointment.id} patient_id=#{patient.id}")
```

```bash
# Find logger calls referencing PHI-shaped models/fields
grep -rniE 'logger\.(info|warn|error|debug).*\.(inspect|to_json|attributes)' --include='*.rb' .
grep -rniE '(logger|console)\.(info|log|warn|error).*\b(patient|member|diagnosis|dob|ssn)\b' --include='*.rb' --include='*.ts' --include='*.js' .
```

Framework-level request logging is an easy miss — Rails logs full param hashes by default unless
filtered:

```ruby
# config/initializers/filter_parameter_logging.rb
# ❌ wrong — default list misses app-specific PHI fields
Rails.application.config.filter_parameters += [:password]

# ✅ correct — PHI fields explicitly filtered from request logs
Rails.application.config.filter_parameters += [
  :password, :ssn, :dob, :diagnosis, :medical_record_number, :insurance_id
]
```

## Exception trackers

Sentry, Bugsnag, Rollbar, and similar tools capture request context (params, headers, session
data) by default — often including PHI the developer never explicitly logged.

```ts
// ❌ wrong — default Sentry config sends full request body, which may include PHI
Sentry.init({ dsn: SENTRY_DSN })

// ✅ correct — PHI-bearing fields scrubbed before the event leaves the process
Sentry.init({
  dsn: SENTRY_DSN,
  beforeSend(event) {
    if (event.request?.data) {
      for (const field of ['ssn', 'dob', 'diagnosis', 'medicalRecordNumber']) {
        delete event.request.data[field]
      }
    }
    return event
  },
})
```

Check for a scrubbing hook (`beforeSend`/`before_send`, `sendDefaultPii: false`) and confirm the
scrub list actually matches this codebase's PHI fields — a copy-pasted generic scrub list that
doesn't name this app's specific fields is a 🟡, not a 🟢. Also confirm the exception tracker
vendor itself is covered by a BAA if PHI can reach it at all (see Gotcha 3 in `SKILL.md`) — a
perfect scrub rule that occasionally misses a new field is still sending PHI to an uncovered
vendor.

## Analytics and telemetry SDKs

Product analytics tools (Segment, Amplitude, Mixpanel, PostHog) and session-replay tools
(FullStory, Hotjar, LogRocket) can capture DOM content, form inputs, or event properties
containing PHI by default.

```ts
// ❌ wrong — full event properties forwarded, including any PHI passed in
analytics.track('Appointment Scheduled', { patientName: patient.name, diagnosis: appointment.reason })

// ✅ correct — only non-PHI identifiers in the event payload
analytics.track('Appointment Scheduled', { appointment_id: appointment.id, clinic_id: clinic.id })
```

For session-replay tools specifically, confirm form-field and text masking is enabled and covers
every PHI-bearing input, not just fields with `type="password"`:

```bash
grep -rniE '(fullstory|hotjar|logrocket|posthog)\.' --include='*.ts' --include='*.js' --include='*.tsx' .
```

A transitive dependency bundling one of these SDKs is a common blind spot — see `SKILL.md`
Gotcha 8; check the resolved dependency tree, not just direct imports in application code.

## LLM prompts and embeddings

Any call to an LLM API is an outbound PHI channel if clinical content reaches the prompt —
including summarization, chatbots, and "AI-assisted" note-taking features.

```ts
// ❌ wrong — raw clinical note sent to a third-party model API with no BAA in place
const summary = await llm.complete(`Summarize this note: ${patient.clinicalNotes}`)

// ✅ correct — either a BAA covers the vendor for this workload, or PHI is stripped/tokenized
//             before the call and rehydrated after, with both facts confirmed, not assumed
const deidentified = stripIdentifiers(patient.clinicalNotes) // see phi-identification.md
const summary = await llm.complete(`Summarize this note: ${deidentified}`)
```

Also check **embeddings and vector stores** — text embedded for semantic search or RAG retains
enough signal to be treated as carrying the same PHI as the source text, and a vector database is
a storage location subject to the same access-control and encryption requirements as any other
PHI store. Don't treat "it's just a vector" as automatically de-identified.

## URLs, query strings, and client-visible errors

Identifiers #14–#15 (URLs, IP addresses) plus general leakage:

```
❌ wrong — patient email in a query string, which lands in server access logs, browser history,
           and any CDN/proxy logging in between
https://app.yourcompany.com/reset?email=jane.doe@example.com

✅ correct — an opaque, single-use token; no PHI in the URL itself
https://app.yourcompany.com/reset?token=8f3d9a1c...
```

Error responses returned to the client should never echo back PHI from the request that
triggered the error:

```ts
// ❌ wrong — validation error echoes the submitted PHI value back in the response body
res.status(400).json({ error: `Invalid SSN format: ${req.body.ssn}` })

// ✅ correct — describes the problem without repeating the sensitive value
res.status(400).json({ error: 'Invalid SSN format' })
```

## Email and SMS

Notification content sent outside the application's own encrypted channels:

```ts
// ❌ wrong — diagnosis detail in a plaintext SMS body
sendSms(patient.phone, `Reminder: your appointment for ${appointment.diagnosis} is tomorrow`)

// ✅ correct — generic notification; details live behind an authenticated portal link
sendSms(patient.phone, `You have an appointment tomorrow. View details: https://portal.yourcompany.com/a/${token}`)
```

Confirm the SMS/email vendor itself is BAA-covered if any PHI does need to travel through it
directly (e.g. a lab result summary in an email body) rather than assuming a generic transactional
email provider is safe by default.

## Seeds, fixtures, and test data

Realistic-looking PHI in `db/seeds.rb`, factory files, or test fixtures is a real exposure if that
data ever reaches a shared dev database, a CI log, or a committed snapshot — and it's also a
repo-hygiene rule this skill itself follows (see `CONTRIBUTING.md`'s no-real-secrets convention).

```ruby
# ❌ wrong — looks synthetic but uses a format indistinguishable from a real SSN/MRN
Patient.create!(name: "Test Patient", ssn: "078-05-1120", mrn: "MRN-100234")

# ✅ correct — obviously synthetic, clearly fake even if leaked
Patient.create!(name: "Test Patient #{Faker::Number.number(digits: 4)}", ssn: nil, mrn: "TEST-000000")
```

Flag any fixture data that looks like it could be a real person's information (a specific,
plausible full name plus a specific plausible DOB/SSN pattern) even if the intent was clearly
synthetic — the goal is data that's unambiguously fake, not merely fictional.
