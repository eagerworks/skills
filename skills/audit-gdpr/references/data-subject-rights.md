# GDPR — Data Subject Rights

> Art 12–22 mapped to code: what "implemented" actually means for each right, and the specific
> ways a real implementation quietly stops short — soft delete instead of erasure, an export
> endpoint that misses half the data, a restriction flag nothing checks.

## Table of Contents

1. [Art 12(3) — the response timeline](#art-123--the-response-timeline)
2. [Art 15 — right of access](#art-15--right-of-access)
3. [Art 16 — right to rectification](#art-16--right-to-rectification)
4. [Art 17 — right to erasure](#art-17--right-to-erasure)
5. [Art 18 — right to restriction of processing](#art-18--right-to-restriction-of-processing)
6. [Art 19 — notifying recipients](#art-19--notifying-recipients)
7. [Art 20 — right to data portability](#art-20--right-to-data-portability)
8. [Access vs. portability — the distinction that gets collapsed](#access-vs-portability--the-distinction-that-gets-collapsed)

---

## Art 12(3) — the response timeline

Every right below shares the same clock: "without undue delay and in any event within one month
of receipt of the request," extendable by up to two further months for complex or numerous
requests, with the extension and its reason communicated within the original month (Art 12(3)).
Art 12(4) requires a denial be communicated within the same one-month window, with reasons and
the right to complain to a supervisory authority and seek a judicial remedy. Art 12(5) makes
fulfillment free by default; a "manifestly unfounded or excessive" request may carry a reasonable
fee or be refused, but that's a narrow exception, not a general throttle.

Audit angle: find the code path that handles a DSAR (data subject access request) or erasure
request and check whether anything enforces this deadline — a ticket queue with no SLA, or a
support inbox with no tracking at all, is a 🟡 at minimum; no request-handling path found at all
for an org clearly processing at scale is a 🔴.

## Art 15 — right of access

The subject can obtain confirmation that their data is processed, and if so, access to the data
itself plus a defined list of metadata (Art 15(1)(a)–(h)): purposes, categories of data,
recipients or categories of recipients (including third countries), the retention period or the
criteria used to set it, the existence of rectification/erasure/restriction/objection rights, the
right to complain to a supervisory authority, the data's source if not collected from the subject
directly, and the existence of any automated decision-making with meaningful information about
the logic involved. Art 15(2) adds: where data has been transferred to a third country, the
subject has the right to be informed of the Art 46 safeguard used. Art 15(3) requires the first
copy be free, in a commonly used electronic format if requested electronically.

An access implementation that returns only the data rows and skips the Art 15(1) metadata block
is incomplete — grade it 🟡, not 🟢, and cite the missing fields specifically.

## Art 16 — right to rectification

The subject can obtain correction of inaccurate data and completion of incomplete data
"including by means of providing a supplementary statement." In code, this is usually the
simplest right to implement (an editable profile) but the easiest to leave incomplete: check that
rectification actually propagates to denormalized copies (search indexes, cached views, exports
already generated) rather than only updating the source-of-truth row.

## Art 17 — right to erasure

Erasure applies when data is no longer necessary for its original purpose, consent is withdrawn
with no other basis remaining, the subject objects under Art 21 with no overriding grounds, the
data was processed unlawfully, erasure is required by law, or it was collected from a child under
Art 8. Exceptions exist (Art 17(3)) for freedom of expression, legal obligations, public health,
archiving/research/statistics, and legal claims — an org can have a legitimate reason to retain
data past an erasure request, but it needs to be one of these, documented, not just "it's easier
not to delete."

**This is the highest-yield check in the whole skill.** A `deleted_at` timestamp that hides a row
from normal queries is not erasure — the personal data is still there, still processed (Art 4(2)
covers storage as processing), and still reachable by anyone with direct database access. A real
erasure flow has to reach:

- The primary datastore (actual deletion or field-level scrubbing, not a soft-delete flag)
- Search indexes (Elasticsearch, Algolia, etc. — a stale document with the person's name is a
  live exposure independent of the primary database's state)
- Caches (Redis, CDN edge caches, application-level memoization keyed by user ID)
- The analytics/data warehouse, if personal data was ever synced there
- Backups — either genuine time-limited retention that expires backups predictably, or a
  documented process to scrub restored backups before they're used
- Art 19: any recipient the data was already disclosed to must be notified of the erasure,
  "unless this proves impossible or involves disproportionate effort" — check whether a
  downstream processor or partner integration gets an erasure webhook/callback at all

```
# ❌ wrong — the row survives everywhere except the default query scope
def erase!
  update!(deleted_at: Time.current)
end
# ✅ correct — reaches storage, index, cache, and logs the downstream notification attempt
def erase!
  transaction do
    update!(name: nil, email: nil, phone: nil, address: nil, erased_at: Time.current)
    SearchIndex.delete_document(id)
    Rails.cache.delete("user:#{id}")
    ErasureNotificationJob.perform_later(id, recipients_notified_of_this_data)
  end
end
```

Grade an erasure endpoint that only touches the primary database 🔴 if the app is known to sync
to a search index, cache, or warehouse — the gap is the exposure, not a nice-to-have.

## Art 18 — right to restriction of processing

A middle ground between rectification and erasure: while accuracy is contested, or the subject
opposes erasure of unlawfully-processed data and wants restriction instead, or the controller no
longer needs the data but the subject needs it for a legal claim, or an Art 21 objection is
pending review — processing (beyond storage) must stop, though the data itself isn't deleted.
Once restricted, Art 18(2) limits further processing to storage, the subject's consent, legal
claims, protecting another person's rights, or important public interest. Look for whether the
app has any concept of "restricted" distinct from active/deleted — most don't, which is itself
worth flagging as a gap rather than assuming it's covered by existing states.

## Art 19 — notifying recipients

Covered above under erasure, but it applies equally to rectification and restriction: any
recipient the data was disclosed to must be told about a rectification, erasure, or restriction,
unless notifying them is impossible or disproportionately effortful. If the subject asks, the
controller must also tell them who those recipients were.

## Art 20 — right to data portability

Narrower than access in scope, but with a distinct output requirement. It applies only when
processing is based on consent (Art 6(1)(a) or Art 9(2)(a)) or contract (Art 6(1)(b)), **and**
carried out by automated means — it doesn't apply to processing based on legitimate interest or
legal obligation, and Art 20(3) explicitly excludes public-task processing. Where it applies, the
subject can receive the data they provided "in a structured, commonly used and machine-readable
format" and have it transmitted directly to another controller "where technically feasible."
Art 20(3) notes this right doesn't affect Art 17 erasure, and Art 20(4) says it can't adversely
affect others' rights (e.g. a joint photo shouldn't be exported with another person's face
un-redacted just because the requester is in it).

## Access vs. portability — the distinction that gets collapsed

Teams routinely build one "export my data" button and call it both. It satisfies neither request
correctly:

| | Art 15 access | Art 20 portability |
|---|---|---|
| Scope | All personal data being processed, plus the Art 15(1) metadata | Only data the subject provided (not data the org derived or inferred) |
| Precondition | None — always available | Only when the basis is consent or contract, and processing is automated |
| Format duty | Copy provided; electronic format if requested electronically | Must be structured, commonly used, machine-readable, and portable to another controller |
| Confirms processing exists | Yes, explicitly (Art 15(1) opening clause) | Not the point of the right |

A single unconditional JSON dump endpoint is a reasonable *starting point* for both, but flag it
as 🟡 if it includes org-derived/inferred fields under a "portability" label (over-scoped for
Art 20) or omits the Art 15(1) metadata block under an "access" label (under-scoped for Art 15).
