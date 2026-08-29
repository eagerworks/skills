# UI States — What Dimensions 5 and 6 Grade Against

Every screen that depends on data or user input has more than one state. A screen is graded on whether each state that *can* happen is designed, not just the happy path. The copyable version to recommend is `assets/ui-states.example.md`.

## The state matrix

| State | When it happens | Must show | Must not |
|---|---|---|---|
| **Loading (initial)** | First render before data | Skeleton matching the final layout, or a spinner in the region (not full-page unless nothing else is useful); appears within 100 ms | Shift layout when data lands; flash a "no results" empty state before data arrives |
| **Loading (refresh / mutation)** | Refetch, submit, pagination | Progress on the control that triggered it (button spinner, row shimmer); existing content stays visible; submit disabled or idempotent | Blank the whole screen; allow double-submit |
| **Empty (first use)** | No data exists yet | Why it's empty + the first action ("No invoices yet — Create your first invoice") | A bare table header or "0 results" |
| **Empty (filtered)** | Search/filter matched nothing | "No results for *X*" + clear-filters action | The same copy as first-use empty |
| **Partial / degraded** | Some data failed, some loaded | The loaded part, plus an inline notice for the failed region with retry | Hide the failure |
| **Error (fetch)** | Request failed | Inline message in the region, plain language, retry action; error boundary at the route so nothing goes blank | Raw error text; console-only; white screen |
| **Error (validation)** | User input rejected | Per-field message next to the field, associated via `aria-describedby`, `aria-invalid="true"`, focus moved to the first error or a summary; input preserved | Toast-only; clearing the form |
| **Error (permission / auth)** | 401 / 403 / expired session | Explain and offer the way forward (sign in again, request access); preserve the intended destination | Generic "Something went wrong" |
| **Success** | Mutation completed | Confirmation the user can perceive (toast with `role="status"`, inline check, redirect + flash) and, for creations, a link to the created object | Silence; a confirmation the screen reader never hears |
| **Destructive confirmation** | Irreversible action requested | Names the object ("Delete *Q3 budget*?"), states consequence, confirm button uses the verb ("Delete"), cancel is the safe default — or skip confirmation and offer **Undo** | `confirm("Are you sure?")`; "OK / Cancel" |
| **Offline** | No connectivity (RN: always; web: PWAs and long sessions) | Banner or inline state; writes queued or clearly blocked; reads served from cache where possible | Indistinguishable from a server error |
| **Long content** | User data longer than designed for | Truncate with ellipsis and full text on hover/focus/expand, or wrap deliberately | Overflow that breaks layout |

## Per screen type — which states are mandatory

| Screen type | Mandatory states |
|---|---|
| List / table / feed | loading, empty (first-use), empty (filtered) if filterable, error, long content |
| Detail / show | loading, error, permission (403), not found (404) |
| Form (create/edit) | validation error (per field), submit pending, success, fetch error if it preloads data |
| Dashboard / composite | loading per region, partial failure, empty per widget |
| Wizard / multi-step | pending per step, data preserved across steps and on error, resumable via URL or storage |
| Destructive action | confirmation or undo |
| Anything on mobile (RN) | offline |

## How to find the evidence quickly

```bash
# Rails
rg -n "\.empty\?|\.any\?|\.none\?" app/views | wc -l              # empty-state branches
rg -n "flash\[|notice:|alert:" app/controllers | wc -l            # success/error feedback
rg -n "data-turbo-confirm|confirm:" app/views | head               # destructive confirmations
rg -n "rescue_from|render.*status: :not_found|:forbidden" app/controllers config | head

# React / Next
rg -n "isLoading|isPending|status === 'loading'|Suspense|loading\.tsx" src app | wc -l
rg -n "isError|error\.tsx|ErrorBoundary" src app | wc -l
rg -n "length === 0|EmptyState|\.length ? " src | wc -l
rg -n "toast\(|useToast|role=\"status\"|aria-live" src | wc -l

# React Native
rg -n "ActivityIndicator|Skeleton|isLoading" src app | wc -l
rg -n "ListEmptyComponent" src app | wc -l
rg -n "NetInfo|useNetInfo|onlineManager" src app | wc -l
rg -n "Alert\.alert" src app | rg -i "delete|remove" | head
```

## Grading shortcuts

| Situation | Grade |
|---|---|
| Primary list screen has no `ListEmptyComponent` / empty branch | 🟡 6.2 (🔴 if the empty state is the *first-run* experience of the product) |
| Route segment fetches without an error boundary / `rescue_from`; a failure renders nothing | 🔴 6.3 on a primary flow |
| Mutation with no pending state on a payment or creation form | 🔴 5.5 |
| Delete via `window.confirm` / plain `confirm:` with generic text | 🟡 6.5 |
| Delete with no confirmation and no undo on a primary object | 🔴 6.5 |
| RN app, writes possible, no `NetInfo`/offline handling | 🔴 6.6 |
| Toast component present but no `role="status"`/`aria-live` | 🟡 6.4 |
| Form re-renders with `f.object.errors` / RHF `errors` next to fields, input intact | 🟢 5.2, 5.6 |
