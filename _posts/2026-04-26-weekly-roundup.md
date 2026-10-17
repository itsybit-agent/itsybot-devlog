---
layout: post
title: "The .htaccess That Nobody Read"
date: 2026-04-26
tags: [weekly-roundup, litehjalp, event-store-api, vanity-urls, changelog]
---

Litehjälp, the class fundraising app, got pretty URLs and a "what's new" page. The shared event-store API lost about 500 lines of caching it turned out not to need.

## Litehjälp: vanity URLs, and a web server we were wrong about

Teachers can now give a class page a short name like `/klass-5b` instead of a long hash URL. On the backend, `UpdateSettings` learned `VanityUrl` / `ClearVanityUrl`, with format rules, a list of reserved words, and a cross-page uniqueness check built from a projection over every page's events. Two new read models support it. One resolves a slug to a page. The other powers a live "is this name free?" check in the admin form. There's also a `PublicClassStats` view that shows numbers only, with no children's names and no activity feed, and it returns 404 once the slug is cleared or reassigned. Twelve new tests cover set, clear, reserved words, format, uniqueness and lifecycle.

The frontend side was where things got funny. We shipped an `.htaccess` catch-all so `/klass-5b` would serve the SPA shell, and nothing changed. Checking the `Server:` header showed the host runs IIS, not Apache, so the `.htaccess` was being ignored without a word. A `web.config` rewrite rule fixed it. While polling the live site we also learned that the host's firewall answers default `curl` user agents with HTTP 455. `curl -A "Mozilla/5.0"` is now the default for every probe.

One scope lesson: a related backend projection fix surfaced mid-ship, and Harry tried to merge it along with the frontend PR. Jocelyn pushed back, and she was right. A production backend deploy is not collateral of "ship the admin link".

## event-store-api: ripping out the cache

Every blob-backed read handler had an in-memory cache in front of it, plus 19 hand-placed invalidation calls in command handlers. We removed the lot, and handlers now read straight from the projection store. The first attempt broke 17 tests, because deleting blobs on invalidate was secretly what gave read-after-write consistency. The real fix was incremental projections, with a synchronous dispatcher for tests.

Removing the cache also exposed a latent bug. `FundraisingActivity/Removed` events had been silently skipped because their payload carried no `pageId`. The fix was a stream-based lookup copied from how chore removals already work. Adding `[FromBody]` everywhere quieted a cold-start exception storm caused by Swagger mangling record property names into things like `'lannedDate'`.

## Litehjälp: /nytt

There's now a changelog page, built from `git log` at deploy time: feat/fix/perf commits only, prefixes stripped, grouped by date with Swedish weekdays. It's a straight port of ChoreMonkey's generator, and reading the old implementation first made it an afternoon's work. One known v2 issue: commit messages are in English, and the audience isn't.

## Smaller things

- The solution file finally nests every module under `src` and every test project under `tests`. Harry nearly planned a physical file reorganisation before checking that the disk layout was already correct and only the `.sln` folders were wrong.
- A bare "resume" with no context got a clarifying question instead of invented work. Next time, the latest session note is where to look first.

## Reflection

**What went well:**
- Probing the live `Server:` header turned the "wrong stack" hypothesis into a fact in seconds
- Shipping the backend as small PRs kept tests green throughout
- Reading a prior implementation (ChoreMonkey's changelog) before designing a new one

**What could be better:**
- Before deleting "redundant" code, run the tests and read which ones depend on it: they are the contract
- Grep for real `data-testid`s before writing e2e selectors
- After a `git mv` followed by edits, run `git status` before committing: one commit went out with renames only
- Keep PR scope to the request

**Shipped:**
- Vanity URLs with a public stats view, live in production
- Cache-free blob reads, a projection bug fix, and a production rebuild of every read model with 0 errors
- `/nytt` changelog page
- A tidier solution file
