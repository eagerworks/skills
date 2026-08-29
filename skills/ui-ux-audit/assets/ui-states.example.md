# UI State Checklist

<!--
Copy this into your project's docs (e.g. docs/ui-states.md) or a PR template.
Fill one table per screen; tick a cell only when the state is designed AND implemented.
Source: the ui-ux-audit skill, references/ui-states.md
-->

Every screen that loads data or accepts input must handle each applicable state below. "n/a" is a valid answer when the state cannot occur — write it, don't leave the cell blank.

## Screen: `<route or component>`

| State | Applies? | Implemented in | What the user sees |
|---|---|---|---|
| Loading (initial) | yes / n/a | `<component or partial>` | Skeleton matching final layout |
| Loading (refresh / mutation) | | | Spinner on the triggering control; content stays |
| Empty (first use) | | | Why + first action ("No projects yet — Create one") |
| Empty (filtered) | | | "No results for *X*" + Clear filters |
| Partial failure | | | Loaded regions + inline retry on the failed one |
| Error (fetch) | | | Plain-language message + Retry; route error boundary |
| Error (validation) | | | Per-field message, `aria-describedby`, input preserved |
| Error (permission / expired session) | | | Explains + way forward; destination preserved |
| Success | | | Toast (`role="status"`) / inline check / redirect + flash |
| Destructive confirmation or Undo | | | Names the object; verb on the confirm button |
| Offline | | | Banner; writes queued or blocked |
| Long content | | | Truncate + full text on focus, or deliberate wrap |

## Shared components to build once

- `Skeleton` / `<%= render "shared/skeleton" %>` — layout-shaped placeholders
- `EmptyState(title, description, action)` — used by every list
- `InlineError(message, onRetry)` — used by every region that fetches
- `FieldError` — associated via `aria-describedby`, rendered under the field
- `ConfirmDialog(objectName, verb)` — never `window.confirm`
- `Toast` with `role="status"` — success and non-blocking errors
- `OfflineBanner` — RN: driven by `NetInfo`; web PWA: `navigator.onLine` + fetch failures
