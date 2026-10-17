---
layout: post
title: "A Template Worth Copying"
date: 2026-08-02
tags: [weekly-roundup, itsybit-todo, event-store-api, one-line-a-day, lantern, kraftskiva]
---

The canary proved apps can live in their own repos. This week a todo list became the template for doing it properly, the host's deploy check stopped racing itself, and Onebit got ready for the Play Store.

*Reconstructed from the commit log — the session notes for this week are gone.*

## Today / Tomorrow: the template app

The canary was the wrong thing to copy: one event, no read model, no frontend. A todo list is about the right size to show the parts that recur in every app: a stream with several event types, a projection folding them into what the UI shows, a command that reads before it writes, and a static frontend talking to the shared host. There's one stream per list, so a read is a single stream read rather than a scan. Events record what happened (`Completed`, `Reopened`) instead of carrying a done flag, and the projection decides what that means now.

The template's docs went through three versions in a day:
- **Eight numbered steps** in dependency order, with the subdomain first. It points out that the frontend and backend paths are independent, and that the two package grants are separate steps because the second can't be done until the first publish succeeds.
- **Restructured.** `src/api` and `src/web` now sit side by side, and the deploy uploads `src/web/` as a whole. An exclude list silently ships anything added later that nobody remembered to exclude, and a directory can't do that. `tests/` holds the projection tests, since that's where the real logic is and it's a pure function.
- **Rewritten as a template doc** rather than a status page. A copied repo shouldn't inherit a table of ticks describing someone else's app.

Two fixes followed, both about what a template teaches:
- **The list ID** lived only in localStorage, so the address bar had nothing to share, and every new browser started a fresh list. A code comment claimed it was "kept in the URL", but the code never did that. The URL is now the source of truth, and there's a share button.
- **Input validation:** the list ID ends up as a directory name in the folder-per-stream store, so it's the one piece of user input that reaches the filesystem. It's now checked against an allowlist and a length limit in all five handlers, reads included. The host sanitises it too, "but a module shouldn't lean on someone else's defence for its own input, and a template least of all."

## event-store-api: the pipeline's first real customers

Registering Todo as the second external module turned up two pipeline bugs:
- **The download redirect:** GitHub Packages answers a download with a 302. curl without `-L` wrote the empty redirect body, exited 0, and the error surfaced two lines later as "not a readable nupkg", blaming the package for a transport problem. The test feed was `file://`, which can't redirect, so a new test serves the feed over HTTP behind a 302.
- **The smoke check race:** the health endpoint answers as soon as *something* is serving, and during a deploy that's still the old build. Phase 6's first good deploy went red reporting the canary missing while it was live seconds later. The check now polls until the expected set appears.

A fix that phase 4 deliberately held back also landed. Bystämma's frontend had never been on the credentialed CORS allowlist, so its realtime hub had been blocked for as long as it existed. Phase 4 found it but left it, because a refactor that also fixed an unrelated bug couldn't be shown to change nothing. A test pinned the broken behaviour with a note to flip it. This flipped it.

At the end of the week, Kräftskiva's song generator could no longer generate at all. Current Claude models reject `temperature` with a 400, and the app sent it on every call. It no longer sends it, and an Opus 5 provider was added.

## Onebit: getting ready for Play

The release setup proven in Vice Versa was ported over: a newer toolchain, release signing, a signed-AAB workflow, and a draft upload to the internal track. Three details came up:
- **The permission:** `USE_EXACT_ALARM` is granted at install, but Play restricts it to alarm-clock and calendar apps, so a journal would have failed review. The app now asks for `SCHEDULE_EXACT_ALARM` at the moment you switch the reminder on, explaining why before it sends you to the settings screen.
- **The app ID** moved to the itsybit namespace before first publication, since it can't change afterwards.
- **A real bug:** every screen captured today's date once, and the process survives overnight. So a line typed the next morning was silently filed under *yesterday*. "Today" now refreshes on resume and whenever the system says the date or timezone changed.

## Lantern

The "so far today" total now includes the running timer, which used to be invisible until you stopped it. That's exactly the number you glance at to decide when to call it a day. There's also a "Tomorrow" on-deck queue: ready-to-start tasks with a tag and a note but no time, which start a live timer when you click one.

## Reflection

**Shipped:**
- Today / Tomorrow as the reference template, registered as the second external module
- Host fixes: download redirects, the smoke check race, Bystämma's hub CORS
- Kräftskiva generating songs again
- Onebit: the Play pipeline, the correct alarm permission, the permanent app ID, and the day-rollover fix
- Lantern: a live total and the on-deck queue
