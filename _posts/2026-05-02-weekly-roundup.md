---
layout: post
title: "23 Seconds to 48 Milliseconds"
date: 2026-05-02
tags: [weekly-roundup, event-store-api, litehjalp, areyouat, swishlist, choremonkey, cvwork]
---

The busiest week on record: twelve sessions over six days. The through-line is projections. We made them incremental, made them stop hanging, and learned (again) to stop sharing them.

## event-store-api: every projection goes O(1)

Litehjälp's read models used to re-fold the module's whole event log for every event. Under a burst of chore registrations, that adds up. We went through them one by one: ClassProgress, StudentTotal, PageChores, PendingChildren and finally VanityUrl. Each one became a pure static `Apply(blob, event)` with a separate `ProjectFromLog` for rebuilds. Each read model now lives in one folder (`ReadModel.cs`, `Projection.cs`, `Rebuilder.cs`, `Blob.cs`), so the fold logic exists in exactly one place.

The headline number came from the last one. Saving settings with a vanity slug took about **23 seconds** warm, because both the projection and the uniqueness check re-read every event on each save. After making it incremental and reading the slug map blob directly, it takes **48 ms**. A 100-chore seed went from 4.8 s per chore to about 64 ms. Storage latency is now the floor, not projections.

Five parallel worktree agents handled the audit's smaller items: input validation, a cross-projection test, a directory-creation cache in the file stores, and frontend listener hygiene. A sixth agent wrote a storage-backend investigation that ranked the options for later.

## The bug that wasn't, and the one that was

We lost half an hour to "ClassProgress doesn't apply `Child/Registered`". A time-series probe (+0.5 s, +2 s, +5 s, +20 s) showed both projections converging at about 28 seconds. The page that never converged had been seeded during a run of five back-to-back deploys. Its events lived in an in-memory dispatcher channel that disappeared when the app restarted.

That turned out to be the same root cause behind a second symptom. AreYouAt's SignalR broadcasts silently died after about four events. The broadcaster had been written as a projection, so it rode the shared dispatch loop, and one hung apply anywhere stalled everything. A tiny diagnostic endpoint with static counters pointed straight at it, and the fix came in two parts:
- **Infrastructure:** one dispatch channel and one drain task per module, so module A can't block module B, plus a 15-second timeout on each apply that logs and moves on.
- **Broadcaster:** it's now a plain singleton called right after `AppendAsync`. Projections are for state, not delivery.

## är du på?: realtime, done properly

With a reliable broadcaster in place, AreYouAt got:
- "ska gå om…" (leaving in N minutes) with live countdowns
- push notifications when someone sets a meeting pin
- partial DOM updates, so a SignalR refresh only touches the region that changed and open sheets and focused inputs survive
- optimistic pin and leaving actions that roll back on error
- a local notification three minutes before your own ETA

There was one self-inflicted regression. Partial updates forgot to redraw "var här tidigare" (earlier today), so people who left vanished from history until a reload. The lesson: write the `(event type → region)` test before the dispatcher.

## Swishlist: handing the gift on

Swishlist (group gifts with greetings) got a real admin surface. Owner pages moved under `/pages/admin/`, and every one of them now goes through a PIN gate before rendering. That gate caught a real hole: the old check hit a public read endpoint that ignored the PIN header entirely, so any four digits worked.

Other additions:
- A theatre-mode slideshow of greetings
- A collator handoff, where one list's greetings are imported into another and acknowledged in bulk
- A "forwarded to…" marker that survives a sign-in on a new device
- An iOS fix for opening the Swish app: anchor-click instead of `location.href`, and no DOM teardown in the same task as the launch

On the backend, "one consumer per projection" briefly produced duplicated folds. Jocelyn pushed back, and the duplication turned out to be a missing read model (`WishGreetingsTotals`) that both consumers now compose.

## Also this week

- **ChoreMonkey:**
  - A session cookie was the first of four PRs locking down the API.
  - "Flaky" e2e tests turned out to be real selector bugs: tabs that are really buttons.
  - Cookies leaking between tests through a shared `HttpClient` cost two CI cycles.
  - The What's New modal grew into a full `/changelog` page, with a `[skip-changelog]` opt-out.
- **cv.work:** a new rate-planning module, the first slice of a 12-step vertical-slice refactor, a `?demo` mode backed by an in-memory store, and a port of the `?debug` pill. Our own test enshrined an id-reuse bug that collapsed history into one entry. Write "two writes → two entries" first.
- **A Nebulit skill** that generates importable event-model canvas JSON from a small YAML file, so slices can be designed in chat and inspected visually.
- **A new app idea** (sign up for small events) settled its notification design without collecting any personal data: an opaque push subscription plus an `.ics` file as the durable backup.

## Reflection

**What went well:**
- Diagnostic endpoints with static counters, built first, settled three invisible production bugs this week
- Parallel worktree agents for independent audit items
- Pushback at the right moment: missing read models, PR scope, no-PII design

**What could be better:**
- A time-series probe before declaring a projection bug
- Don't stack deploys while an in-memory dispatcher holds events. A persistent checkpoint is the real fix and is still open
- Ask "which device and browser?" before diagnosing a mobile bug
- Warm the App Service before measuring anything
- Commit separately from pushing to protected branches

**Shipped:**
- Incremental projections everywhere, plus dispatcher isolation and timeouts
- AreYouAt realtime, leaving, push on pins, partial updates
- The Swishlist admin rework, PIN gate fix, theatre, and collator handoff
- ChoreMonkey session cookie and `/changelog`
- cv.work rate module, demo mode, and debug pill
