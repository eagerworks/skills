# Repo Handoff — Dimensions, Checks, and Question Bank

This file is the single source of truth for what the analysis inspects, how each check is graded, and which question is asked when a check is not 🟢. `SKILL.md` only summarizes it.

## Grade ladder

| Grade | Rule |
|---|---|
| 🟢 Documented | The repo answers the check **and** the code agrees with what's written. Cite `file:line` or the command output. |
| 🟡 Partial | Written down but incomplete, outdated, or contradicted by the code. Cite the contradiction. |
| 🔴 Missing | Not in the repo, and the system can't be run, shipped, or operated safely without it. |
| ⚪ Unverifiable from code | Lives in a console, dashboard, contract, chat, or person. Always produces a question. |

**Conservatism rule:** when in doubt between two grades, choose the one that produces a question. A wrong 🟢 silently loses knowledge; a wrong 🟡 costs the previous team thirty seconds.

**Question rule:** a question is emitted only for a check graded 🔴, 🟡, or ⚪, and it must name the evidence gap ("`README.md` describes Heroku but `fly.toml` exists — which is live?"). Questions from the bank below are templates; adapt them to what you actually found.

## Priority ladder for questions

| Priority | Meaning |
|---|---|
| **P0** | The receiving team cannot run, deploy, or keep the system alive without the answer. Must be answered before the previous team leaves. |
| **P1** | Needed within the first month: to ship changes confidently, to debug production, to plan work. |
| **P2** | Context and history: nice to know, improves decisions, not blocking. |

Checks below carry a default priority; raise it when the evidence warrants (an undocumented payment provider is always P0).

---

## 1. Overview & architecture

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 1.1 | Purpose and users are stated | README or docs say what the system does and for whom | P1 |
| 1.2 | Component map exists | Services, apps, packages, and their boundaries are listed (monorepo layout, service list, diagram) and match the directory tree | P1 |
| 1.3 | Key domain models are identifiable | Core entities (schema, models, types) are named and their relationships are discoverable | P1 |
| 1.4 | Architectural decisions are recorded | ADRs, design docs, or dated decision notes exist for non-obvious choices (framework swaps, custom auth, event bus) | P2 |
| 1.5 | Non-obvious code has intent | Custom frameworks, generated code, vendored libs, and "magic" directories are explained somewhere | P2 |

**Question bank**
- What are the top 3 things this system must never do wrong (money, data loss, downtime)? Who is affected when it fails?
- Which parts of the codebase are considered "done and stable" vs. "in flux" or half-migrated?
- Which components are still in use and which are dead but not deleted?
- Were there major rewrites or abandoned migrations? What was left behind?
- Are there any external consumers of this system's API/data we can't see from the code?

## 2. Environment & setup

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 2.1 | Setup is documented and works on paper | A single documented path (`bin/setup`, `make setup`, README steps) exists and every referenced script/file exists | P0 |
| 2.2 | Toolchain is pinned | `.ruby-version`, `.node-version`/`.nvmrc`, `.tool-versions`, `engines`, `python-version`, Dockerfile base image | P0 |
| 2.3 | Lockfiles are committed | `Gemfile.lock`, `package-lock.json`/`pnpm-lock.yaml`/`yarn.lock`, `poetry.lock`, `go.sum` | P0 |
| 2.4 | Required env vars are enumerated | `.env.example` / `config/credentials` template lists every variable the code reads (grep `ENV[`, `process.env.`, `os.environ`) | P0 |
| 2.5 | Local services are declared | Databases, queues, caches required locally are in `docker-compose` or the setup doc | P0 |
| 2.6 | No hidden manual steps | No references to "ask X for the file", VPN, private registries, or pre-seeded data without instructions | P0 |

**Question bank**
- Is there a working `.env` for development you can hand over (with placeholders where secrets must be rotated)?
- Are there private packages/registries, VPNs, or licensed tools required to build? Who administers them?
- Which OS/machines does the team develop on? Any known setup gotchas not in the README?
- Is there a seed/fixture dataset for local development? Where does it come from?

## 3. Build, test & quality

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 3.1 | Build/lint/typecheck/test commands exist | Each is a real script/target, documented, non-interactive | P0 |
| 3.2 | The suite passes today | Ran once under the rules in `handoff-workflow.md`, exit 0 — or ⚪ if it couldn't be run | P1 |
| 3.3 | Test surface matches the risk | Tests exist where the money/data-critical code lives (payments, auth, billing, migrations) | P1 |
| 3.4 | Known-flaky or skipped tests are marked | `skip`, `xit`, `pending`, `@flaky`, `retry` annotations are explained | P1 |
| 3.5 | Quality gates are in CI | Lint/test run on PRs and their failure blocks merge | P1 |

**Question bank**
- Which tests are known to be flaky, slow, or skipped, and why?
- Which parts of the system have no automated coverage and are verified manually? How?
- Is there a QA/staging checklist before a release?
- Are there tests that require external services or production-like data to run?

## 4. Infrastructure & deploy

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 4.1 | Hosting is identifiable | Provider and topology are stated and match the config in the repo (Dockerfile, `fly.toml`, `kamal`, Terraform, `serverless.yml`, k8s manifests) | P0 |
| 4.2 | Deploy procedure is documented and matches the code | The described command/pipeline exists and targets the identified hosting | P0 |
| 4.3 | Environments are listed | dev / staging / prod (and any others) with URLs and how config differs | P0 |
| 4.4 | Rollback is documented | How to revert a bad deploy, including DB migrations | P0 |
| 4.5 | Infrastructure is versioned | IaC in the repo, or an explicit statement that it's click-ops (⚪) | P1 |
| 4.6 | Domains, DNS, TLS are identified | Where the domain is registered, who manages DNS, how certs renew | P0 |

**Question bank**
- Where exactly does production run (accounts, regions, projects, clusters)? Who owns those accounts?
- Walk us through the last production deploy step by step. What can go wrong?
- How do you roll back? Has it ever been done?
- Which environments exist, what are their URLs, and which are safe to break?
- Where is the domain registered, who manages DNS, and when does the domain/certificate renew?
- Is any infrastructure managed by hand (not in code)? What would we lose if the cloud account were recreated?
- Are there any IP allow-lists, firewall rules, or partner VPN tunnels we must preserve?

## 5. Data

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 5.1 | Datastores are enumerated | Every DB, cache, queue, search index, object store the code connects to is listed | P0 |
| 5.2 | Migrations are consistent | Migration files form a linear history; no evidence of hand-applied changes (`schema.rb`/`prisma/schema` matches migrations) | P1 |
| 5.3 | Backups and restore are documented | Frequency, location, retention, and a tested restore procedure | P0 |
| 5.4 | PII and sensitive data are identified | Which tables/fields hold personal, payment, or health data; encryption at rest | P1 |
| 5.5 | Retention and deletion are defined | Policies or jobs for purging data; regulatory constraints (GDPR, HIPAA, PCI) | P1 |
| 5.6 | Data volume is known | Row counts / storage size / growth rate — almost always ⚪ | P1 |

**Question bank**
- What is the size of production data, and how fast does it grow?
- Where are backups, how often are they taken, when was a restore last tested?
- Are there migrations that were applied manually, skipped, or are dangerous to re-run?
- Which fields contain PII or regulated data? Are there data-processing agreements we inherit?
- Are there data feeds in or out (ETL, exports, partner syncs, analytics warehouses) not visible in this repo?

## 6. Third-party services & credentials

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 6.1 | Every external service is enumerated | Grep SDKs, API hosts, env var names; each appears in a service inventory in the docs | P0 |
| 6.2 | Account ownership is stated | For each service: which org/email owns the account, billing owner | ⚪ → P0 |
| 6.3 | Secret storage is identified | Where runtime secrets live (vault, cloud secret manager, CI secrets, `.kamal/secrets`, encrypted credentials) | P0 |
| 6.4 | Rotation on handoff is planned | Which credentials must be rotated when the previous team loses access | P0 |
| 6.5 | Webhooks and callbacks are listed | Inbound URLs registered at third parties that must be re-pointed if hosting changes | P1 |
| 6.6 | Plans, quotas, and contracts are known | Paid tiers, rate limits, renewal dates — ⚪ | P1 |

**Question bank** (ask per service found)
- For **<service>**: which account/email owns it, who is the billing contact, and can ownership be transferred to us?
- Where does the production credential for **<service>** live today, and who has access to it?
- Which credentials will you rotate before leaving, and which do we need to rotate ourselves?
- Are there webhooks/callbacks registered at **<service>** pointing to our infrastructure? Where are they configured?
- What plan/tier are we on, what does it cost, and when does it renew?
- Are there sandbox/test accounts we should receive too?

## 7. Security & access

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 7.1 | Auth model is documented | How users/services authenticate and authorize; roles; SSO/OAuth providers | P1 |
| 7.2 | No secrets in the working tree | `.env`, key files, tokens in config are absent or gitignored | P0 |
| 7.3 | No secrets in git history | Read-only scan (`git log -p` grep for known patterns, or a secret scanner in read-only mode) is clean, or findings are listed with commit hashes only | P0 |
| 7.4 | Dependency vulnerabilities are known | Read-only audit (`npm audit`, `bundle audit check`, `pip-audit`) run or ⚪; critical findings listed | P1 |
| 7.5 | Access inventory exists | A list of every system a person needs access to (repo, cloud, DNS, stores, SaaS, monitoring) | P0 |
| 7.6 | Admin/superuser paths are identified | Admin panels, feature flags, impersonation, console access — and who has them | P1 |

**Question bank**
- Please list every system where someone on your team currently has access that we will need: repo hosting, cloud, DNS, app stores, SaaS dashboards, monitoring, email, password manager.
- Which of those accounts are personal (tied to an individual's email) rather than organizational?
- Have there been security incidents or known vulnerabilities? Any pending remediation?
- Are there compliance obligations (SOC 2, GDPR, PCI, HIPAA) with audits or evidence we inherit?
- Who can access production data today, and how?

## 8. Code health & known debt

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 8.1 | Hot spots are identified | `git log --format=%H --name-only` churn ranking; the top files are explained or clearly core | P2 |
| 8.2 | TODO/FIXME/HACK density is measured | Count and cluster; anything referencing a person, ticket, or date is listed | P2 |
| 8.3 | Dead code signals | Unreferenced modules, commented-out blocks, feature flags always off, routes with no handlers | P2 |
| 8.4 | Dependencies are current enough | Read-only outdated check; majors behind and EOL runtimes listed | P1 |
| 8.5 | Known debt is written down | A debt list, a "gotchas" doc, or issues labelled tech-debt | P1 |

**Question bank**
- What would you refactor first if you had another month? What do you regret?
- Which files or modules are scary to touch, and why?
- Are there upgrades you postponed (framework, runtime, major deps) and what blocked them?
- Which TODOs are real and which are safe to ignore?

## 9. Process & history

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 9.1 | Branching and release model are stated | Documented and consistent with `git branch -a`, tags, and PR history | P1 |
| 9.2 | Bus factor is measured | `git shortlog -sn --no-merges` over the last 12 months; concentration reported as fact | P0 if one author dominates |
| 9.3 | Unfinished work is inventoried | Unmerged branches with recent commits, open PRs, WIP commits | P1 |
| 9.4 | Issue tracker is linked | Where work is tracked; open issues count; anything labelled urgent/blocked | P1 |
| 9.5 | Activity timeline is known | Last commit date, cadence over time, obvious gaps | P2 |

**Question bank**
- Who wrote most of this, and will they be reachable after the handoff? For how long, and through which channel?
- What is in progress right now? Which branches/PRs should we finish, and which should we discard?
- Where is the backlog, and which items did the client/stakeholders consider committed?
- Were there other repositories, forks, or internal tools that are part of this system?
- What was the release cadence, and who decided when to ship?

## 10. Operations & support

| # | Check | 🟢 when | Default priority |
|---|---|---|---|
| 10.1 | Monitoring and error tracking are identified | SDKs/config for APM, error tracking, uptime checks are present and their dashboards named | P0 |
| 10.2 | Logging is described | Where logs go, retention, how to search them | P1 |
| 10.3 | Alerting and on-call are described | Who gets paged, through what, for which conditions | P0 |
| 10.4 | Runbooks exist | Procedures for the common failure modes (queue backed up, DB full, cert expired) | P1 |
| 10.5 | Scheduled jobs are enumerated | Cron, `whenever`, `sidekiq-scheduler`, cloud schedulers, GitHub Actions on `schedule:` — each with purpose and safe-to-rerun status | P0 |
| 10.6 | Support channels and SLAs are known | How users report problems; response commitments — ⚪ | P1 |

**Question bank**
- What breaks most often, and what do you do when it does?
- Which scheduled jobs exist, what happens if one is skipped or run twice, and which are safe to disable?
- Where do alerts go today, and who is on the receiving end? Will that stop working when you leave?
- Are there recurring manual operations (monthly invoicing run, data cleanup, cert renewal)?
- Are there SLAs or support commitments with the client or end users?
- Is there anything scheduled or expiring in the next 90 days (certs, domains, tokens, contracts, app-store reviews)?
