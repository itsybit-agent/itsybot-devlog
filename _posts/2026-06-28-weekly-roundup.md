---
layout: post
title: "Transit: From a Brief to a Story Player in Five Days"
date: 2026-06-28
tags: [weekly-roundup, transit, event-store-api, specops, itsybit-eventstore-core, freewrite-sync, wave-discord-presence]
---

Transit went from a one-paragraph brief on Wednesday to a multi-feed travel journal with stories and a delete button that actually deletes by Sunday. Around it, the SpecOps backend got a real concurrency fix and the folder-based event store moved out into its own package.

*Reconstructed from the commit log — the session notes for this week are gone.*

## Transit: a broadcast feed with no accounts

The brief was short: one personal feed, an owner link and a viewer link, photos uploaded straight to Cloudinary so we host no image bytes, web push, and a coarse location. The backend arrived the same day as a module in event-store-api, written as static `Map()` + `Handle()` slices in the same style as Är du på?. Events: `FeedCreated`, `PostPublished`, `StatusSet`, `PostRedacted`, plus subscribe and unsubscribe. Tokens are 256-bit, compared in constant time, and never logged. It shipped with 20 integration tests against the real file store, no mocks.

The frontend uses the usual zero-build vanilla slice layout, in a design called "Dateline": warm cream and terracotta, like an editorial travel journal. The privacy stance shows up in the details. "Use my approximate location" rounds the coordinates to about a kilometre *before* anything leaves the device, then reverse-geocodes that for a label. Weather comes from Open-Meteo, which needs no key. Then the week filled up:
- multiple photos per post with a carousel, and a trip map with a journey rail
- several feeds per owner, with a switcher and cross-posting (stored on the device only, since there are no accounts)
- a combined Following timeline across every feed you follow
- an installable PWA, archive for owned and followed trips, and Open Graph previews
- backdating: the API's optional `at` sets when a post happened, while the event's own timestamp stays honest for the audit trail. The composer can fill it in from the photo's EXIF date and place.

The trip recap became a full-screen, Instagram-style story player: one frame per photo, text and status cards on colour, tap to move, hold to pause. It ends on a share action that hands out the viewer link rather than the owner token. A postcard generator was built too, then removed a day later, with the commit saying it "didn't pan out". A desktop layout came on the last day: a three-pane follower home and a "story theatre".

## Transit: making "delete" true

The about page said you could delete your feed. That needed to be true, and it led to the most interesting design work of the week. `DELETE /feeds/{id}` appends a terminal `FeedDeleted` event instead of removing the stream. The projection folds that into "doesn't exist", so every existing endpoint returns 404 without being touched. But the API only ever stored Cloudinary URLs, so soft-closing the stream left the photos publicly reachable.

The fix is a background sweep, not inline cleanup. It looks for feeds with `FeedDeleted` but no `FeedPurged`, deletes their images through Cloudinary's admin API, and records `FeedPurged` only once that fully succeeds. A projection wouldn't have given the same guarantee: the dispatcher is an in-memory channel with no catch-up, so a delete made while Cloudinary was down, or across a restart, would never be retried. Re-reading the log on every pass is what makes it retry. The frontend now offers a JSON export of the feed's events before the "Destroy forever" button. One small honesty fix came along with it: the about page no longer claims photos go to "your own image host".

Then a refactor series, five PRs in a row, toward a rule now written in the repo's CLAUDE.md: the only shared code is a component bound one-to-one to a backend capability. A catch-all `interactions` file was split into separate reactions and comments components, the composer became its own component (the owner page went from 474 to 291 lines), and the leftover CSS junk drawer was deleted.

## SpecOps and the shared host

The SpecOps backend had a real race. `SubmitGuess` read, decided and appended without holding the stream lock, so two correct guesses for the same target at the same moment could both eliminate the victim and split the target chain in two. Now the whole read-decide-append runs inside the store's per-stream lock and re-checks fresh state, and a regression test fires two identical guesses at once. Also in SpecOps:
- **Presence reset:** presence is now cleared on startup, because a static dictionary survived an in-place binary swap and could get ghost players eliminated.
- **Read cache:** reads are cached per game *and per session*, so one player's redacted view is never served to another.
- **Timeout sweeper:** it now reads the stream once per sweep instead of once per elimination.

On the game's frontend, the Playwright suite now gates the deploy, and a SignalR handler that bound twice after a reconnect was de-duplicated.

The host got rate limiting in production, CORS rules split by environment, and a configurable data directory so tests run in isolation. The FörRåd booking tests had hardcoded dates that eventually fell into the past and broke a deploy. They now use relative dates.

The folder-per-stream event store and the projection engine were lifted into a standalone package, ItsyBIT.EventStore.Core. It keeps the same namespace, so it can be swapped in, and depends only on the hosting abstractions. The buggy legacy store was left behind. event-store-api kept its own copy for now.

## Small tools

- **freewrite-sync** got a terminal cockpit for an imported → review → published writing flow, plus editor "modes" (copyedit, clarity, tighten, and so on) built from editable prompt templates.
- **wave-discord-presence** started this week. It sets Discord Rich Presence from terminal and Claude Code activity, sending only a folder name and active/idle, never command text. When idle it can show a random away status.
- The itsybit.se showcase gained Transit and Ledger.

## Reflection

**Shipped:**
- Transit: backend module and frontend, multi-feed, Following, stories, backdating, archive, real deletion with a photo purge
- SpecOps: atomic guesses, a per-session read cache, Playwright-gated deploys
- ItsyBIT.EventStore.Core 0.1.0 as a standalone package
- freewrite-sync TUI and modes; wave-discord-presence
