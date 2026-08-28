# Monorepo & EAS — Finding the App and Getting It Submitted

> Where the review-relevant config lives in a Turborepo, and the build/submit-time gates specific
> to a monorepo + EAS setup rather than to store policy itself.

## Table of Contents

1. [Locating mobile apps in a Turborepo](#locating-mobile-apps-in-a-turborepo)
2. [Why monorepo static analysis goes wrong](#why-monorepo-static-analysis-goes-wrong)
3. [Metro config for monorepos](#metro-config-for-monorepos)
4. [eas.json profiles](#easjson-profiles)
5. [.easignore and what gets uploaded](#easignore-and-what-gets-uploaded)
6. [Versioning](#versioning)
7. [Credentials](#credentials)
8. [Turborepo task wiring](#turborepo-task-wiring)

---

## Locating mobile apps in a Turborepo

1. Read `turbo.json` for tasks that look mobile-shaped: `build`, `ios`, `android`, `eas-build`, or a filtered `build#apps/mobile`-style pipeline entry.
2. Read the workspace glob — `pnpm-workspace.yaml` (`packages:` list), or `package.json#workspaces`, or `turbo.json#packages` — to enumerate candidate package directories, typically `apps/*`.
3. Within each candidate, check for `app.json`/`app.config.*`, an `expo` dependency, or `ios/`+`android/` dirs (see `expo-and-rn.md` § Workflow classification).
4. **A monorepo can contain more than one mobile app** (e.g. a consumer app and an internal/admin app, or a whitelabel setup with multiple `apps/*-mobile` packages sharing code). Audit each independently — don't merge findings, and note in the report if a shared `packages/*` dependency causes the same finding to repeat across apps (fix it once, upstream).

## Why monorepo static analysis goes wrong

- **Hoisting.** With npm/Yarn classic hoisting (less common with pnpm, which is stricter), a dependency can be resolved from the repo root `node_modules` rather than the app's own — grepping only `apps/mobile/node_modules` misses it. Prefer `npx expo config`/`expo-modules-autolinking` (which resolve the real module graph) over manual `node_modules` greps.
- **Symlinked workspace packages.** `packages/analytics`, `packages/ui`, etc. are symlinked into the app's dependency tree by the package manager. A permission-carrying native module can be pulled in transitively through one of these — see `permissions-and-privacy.md` § Common false 🟢.
- **A single `package.json` audit is not the whole picture.** Always resolve the full dependency graph for the specific app being audited, not just its direct `dependencies` list.

## Metro config for monorepos

Not a store-policy issue, but a **build blocker** worth flagging if broken, since a broken build means the app can't be submitted at all:

```js
// ✅ correct — metro.config.js aware of the monorepo root
const { getDefaultConfig } = require('expo/metro-config')
const path = require('path')

const projectRoot = __dirname
const workspaceRoot = path.resolve(projectRoot, '../..')
const config = getDefaultConfig(projectRoot)

config.watchFolders = [workspaceRoot]
config.resolver.nodeModulesPaths = [
  path.resolve(projectRoot, 'node_modules'),
  path.resolve(workspaceRoot, 'node_modules'),
]

module.exports = config
```

Check for `watchFolders` including the monorepo root, `nodeModulesPaths` covering both the app-local and root `node_modules`, and (if using pnpm, which is stricter about hoisting) `unstable_enableSymlinks: true` and `disableHierarchicalLookup` set deliberately, not by default confusion. A missing `watchFolders` entry usually surfaces as a Metro "unable to resolve module" error referencing a workspace package — flag this as a build blocker, not a store-policy finding.

## eas.json profiles

```jsonc
{
  "build": {
    "development": { "developmentClient": true, "distribution": "internal" },
    "preview": { "distribution": "internal", "channel": "preview" },
    "production": { "autoIncrement": true, "channel": "production" }
  },
  "submit": {
    "production": { "ios": { "appleId": "you@yourcompany.com" }, "android": { "serviceAccountKeyPath": "./service-account.json" } }
  }
}
```

Identify which `build` profile is production (usually named `production`, but confirm — some teams name it `release` or point their CI at a differently-named profile). Everything in `references/audit-workflow.md` Phase 4/5 should be checked against **that** profile's resolved env, not `development`/`preview`. Check for:
- **Env leakage** — a production profile inheriting a `.env` meant for development (staging API URLs, debug flags), or `env` keys hardcoded per-profile that weren't updated when a new env var was added elsewhere.
- **`channel`/`distribution`** — production should be `distribution: "store"` (implicit default) with a distinct `channel` from preview/development, so OTA updates don't leak between them.

## .easignore and what gets uploaded

EAS Build uploads the project as a tarball respecting `.easignore` (falls back to `.gitignore` if absent). In a monorepo, a missing or wrong `.easignore` can either bloat the upload with unrelated `apps/*`/`packages/*` content, or — more seriously — exclude a workspace package the app actually needs, causing a build failure that looks unrelated to the missing file. Not a store-policy issue directly, but flag if `.easignore` looks like it would exclude a directory the app imports from.

## Versioning

- **`autoIncrement`** in the production build profile increments `ios.buildNumber`/`android.versionCode` automatically per build — confirm it's on, or that some other mechanism (CI step, `eas build:version:set`) reliably increments it, since a non-incrementing build number is a **hard submit-time rejection** on both stores.
- **Remote vs. local version source** — `cli.appVersionSource` in `eas.json` determines whether EAS or the local `app.config.ts`/native files are authoritative for the version. Check this matches the team's actual workflow (if set to `"remote"` but the team also hand-edits `app.config.ts`'s version, the two can drift).

```bash
npx eas build:version:get      # read current version/build numbers without changing them — safe
```

## Credentials

Read-only inspection only:

```bash
npx eas credentials             # interactive; use list/view options, don't generate or revoke
```

Look for: an iOS distribution certificate/provisioning profile that's expired or about to expire, entitlements requested in `app.config.ts` (e.g. push notifications, associated domains) without matching capabilities enabled in the provisioning profile (or vice versa — a capability enabled but unused, which is harmless but worth a note), and — for Android — a Play service-account JSON referenced in `eas.json#submit` that should never be committed to the repo (check `.gitignore` covers it; if the file itself is present in the repo, that's a credential-leak finding, not a store-review one, but worth flagging immediately and separately from the graded report).

## Turborepo task wiring

Confirm the mobile app's build task is correctly declared in `turbo.json` with the right `dependsOn`/`outputs` so that a `turbo build --filter=apps/mobile` (or equivalent) picks up changes to workspace packages it depends on — a stale cache serving an outdated `packages/api-client` into an EAS build is a subtle, hard-to-diagnose build issue, not a review issue, but worth a note if the pipeline config looks likely to cause it (e.g. missing `outputs` on a shared package's build task).
