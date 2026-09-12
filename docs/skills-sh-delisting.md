# Removing deprecated skills from the skills.sh listing

This repo has no control over this — it's a limitation of the third-party
[skills.sh](https://skills.sh) website (built and run by the `vercel-labs/skills`
project), not something `eagerworks/skills` can fix from its own side. This doc
exists so the next person who hits it doesn't have to re-research it.

## What we control vs. what we don't

- **The `npx skills` CLI** (`add`, `list`, `find`, etc.) always reflects the
  live contents of this repo. Deleting a skill's `skills/<name>/` directory
  here is immediately correct as far as the CLI is concerned — a deleted skill
  will not install and will not show up in `--list`.
- **The skills.sh website's listing page** for a repo is a different thing: it's
  built from accumulated install telemetry, not a live sync of the repo tree.
  A skill removed from `skills/` here can keep showing up there — with its old
  install count — indefinitely, because nothing re-derives the listing from the
  current repo state.

## Why deleting the folder isn't enough

There is currently **no documented, self-serve way for a repo owner to delist a
stale skill from the skills.sh website**. This is confirmed by an open,
unresolved upstream issue:

- [vercel-labs/skills#1578 — "Repo page lists stale skills that no longer exist in the repo - how can an owner delist them?"](https://github.com/vercel-labs/skills/issues/1578)

That issue reports the same three symptoms we should expect here if we prune
skills:
1. Deleted skills keep their install telemetry and stay listed.
2. Skills marked `metadata.internal: true` in `SKILL.md` frontmatter (intended
   to hide a skill) are not reliably hidden on the website — only in the CLI.
3. The CLI and the website disagree: the CLI is correct (reflects the repo),
   the website is not (reflects telemetry history).

## What actually works today

Based on multiple Vercel Community threads where repo owners hit this and got
it resolved, the current workaround is to ask the Vercel team directly — there
is no ticket/form, just posting in the community forum:

- [Remove obsolete skills from Apify listing on skills.sh](https://community.vercel.com/t/remove-obsolete-skills-from-apify-listing-on-skills-sh/48381)
- [Remove obsolete skill from skills.sh list](https://community.vercel.com/t/remove-obsolete-skill-from-skills-sh-list/45686)
- [Removing a skill from the skills.sh list](https://community.vercel.com/t/removing-a-skill-from-the-skills-sh-list/35562)
- [How to remove or update obsolete skill lists on skills.sh](https://community.vercel.com/t/how-to-remove-or-update-obsolete-skill-lists-on-skills-sh/37935)
- [skill.sh listing incorrect](https://community.vercel.com/t/skill-sh-listing-incorrect/44023)
- [Why is a deleted skill still listed on skills.sh and new ones not showing in search?](https://community.vercel.com/t/why-is-a-deleted-skill-still-listed-on-skills-sh-and-new-ones-not-showing-in-search/38998)

## Recommended process for this repo

When a skill in this repo is deprecated:

1. Remove `skills/<name>/` and its parallel `evals/<name>/` from the repo, and
   drop the row from the **Available skills** table in `README.md`, as a normal
   commit (see the root `CLAUDE.md` commit-type conventions).
2. If the skill still shows up on `skills.sh/eagerworks/skills` afterward, post
   in the [Vercel Community forum](https://community.vercel.com/) (or add a
   comment on [#1578](https://github.com/vercel-labs/skills/issues/1578) if
   it's still open) naming the exact skill and repo, and ask for it to be
   removed from the listing. There's no faster path today — check the issue
   first in case the team has since shipped a self-serve delist mechanism.
