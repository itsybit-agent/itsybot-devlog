---
layout: post
title: "Six Phases and a Canary: Apps That Live in Their Own Repos"
date: 2026-07-26
tags: [weekly-roundup, event-store-api, itsybit-eventstore-core, one-line-a-day, viceversa, lantern, bystamma]
---

The shared API host spent the week being taken apart, carefully, so new apps no longer have to live inside it. Meanwhile, Onebit's 9 PM reminder kept arriving in the middle of the night, and it took three commits to explain why.

*Reconstructed from the commit log — the session notes for this week are gone.*

## event-store-api: the pluggable-apps plan

Until this week, every app's backend was a module compiled into one monorepo host. The plan was written as phases in `docs/pluggable-apps-plan.md`, and each phase was shipped so it could be undone on its own: one variable per deploy.

- **Phase 0, safety net.** Nightly data backups, and a smoke check after deploy that polls the health endpoint, then asserts that the debug page lists every expected module. Module discovery fails silently, so before this an app could quietly vanish after a deploy and the run would still be green. The first live backup failed with a 401: basic auth is disabled on the platform's file API by default for newer apps, so it switched to an Azure AD bearer token.
- **Phase 1, the contract package.** ItsyBIT.EventStore.Core grew everything the host and modules compile against: `IModule`, CORS policies, the ID generator, and the legacy file store, so swapping it in changes nothing about storage. An audit found that all the previously shared files were already identical.
- **Phase 2, the swap.** The host, all 13 modules and their tests moved from the in-repo project to the published package. That deploy failed at restore with a 403: an explicit `permissions:` block cuts the built-in token down to exactly what's listed, so the package access grant also needed `packages: read`. While fixing it we found that two test projects had incomplete solution configuration entries, so solution-level builds, CI included, had been silently skipping them: **316 tests that never ran in the pipeline.** With them back, the suite is 733 tests across 10 projects, all passing.
- **Phase 3, storage routing.** A routing store sends each module to either the folder-per-stream store or the legacy one, based on a config list that ships empty. So the deploy changes nothing, and older apps never write the folder format, which keeps every later rollback data-safe.
- **Phase 4, modules declare themselves.** The host had one hardcoded path check per module to decide which ones skip the API key, plus a flat list of credentialed CORS origins, and an app in its own repo can edit neither. `IModule` gained `RoutePrefix`, `RequiresApiKey` and `CorsOrigins`. `RequiresApiKey` defaults to true, so a module that forgets to declare it fails closed.
- **Phase 5, external modules.** The deploy pulls modules listed in `modules.json` from the private package feed and drops them next to the host. Discovery picks them up, so the drop *is* the install. The app repo never holds cloud credentials: it publishes a package and sends a dispatch event to ask for a deploy. Deploys now queue rather than cancel, because cancelling mid-upload is how you get a half-written site.
- **Phase 6, the canary.** The first app that lives in its own repo is deliberately trivial: record a ping, read it back. If the ping doesn't come back, the plumbing is broken, not the app. It is also the first real module on folder storage.

The canary's own first release had two snags. GitHub rejected the workflow with no logs at all, because the `secrets` context isn't allowed in a step-level `if:`. Then its restore failed with a 403: GitHub Packages scopes a package to the repo that published it, so the canary's built-in token couldn't read the contract package. A personal access token fixed it, but by Monday that was walked back in the docs. Granting the app repo Actions access on the package is the proven path, and the host had been working that way all along. There is a UI trap here too: the package page has both "Manage access" and "Manage Actions access", and searching the first one for a repo name finds nothing.

## Onebit: the reminder that drifted past midnight

The daily reminder was a WorkManager periodic job. Doze batches those, so the 9 PM nudge landed whenever the phone next woke up, often just after midnight. The first fix moved to `AlarmManager.setAndAllowWhileIdle`. The nudge then arrived at 3:58 AM. That call gets past Doze, but it's still an *inexact* alarm, and App Standby holds inexact alarms for an app that's opened for a few seconds a day. Only `setExactAndAllowWhileIdle` escapes both, so it now uses that, with a fallback and a one-tap hint if exact-alarm access is ever revoked.

There was a third bug hiding under the drift. The "have you already written today?" check used the date the alarm *arrived*, so a midnight delivery nagged someone who'd written the evening before. The alarm now carries the day it was set for, and a late delivery stays silent. History photos also became squares instead of a cropped 3:1 strip.

## Vice Versa: logging without opening the app

Google Play flagged the app's target API level, so the phone and watch apps both moved to Android 16. CI version codes now start from an offset above the codes that had been uploaded by hand, which would otherwise have been rejected. The new feature is quick logging: a Glance home-screen widget shows today's tasks as tap-to-log pills plus the balance. It appends to the same event store and folds through the existing projections, so no new read model was needed. The watch tile went from two tall chips to four compact ones, with unlogged tasks first.

## Smaller things

- **Lantern:** when a scheduled block's time arrives and no timer is running, the focus card shows it as "active now" with a one-tap "Begin this block", instead of looking idle.
- **Bystämma:** a voter activation landing screen was merged.
- **itsybit.se:** the CV.GEN and cyoa "Try it" links now point at their actual subdomains.

## Reflection

**What went well:**
- Shipping one change per deploy made every failure easy to pin on one cause, including a CI gap that had hidden 316 tests

**Shipped:**
- Pluggable-apps phases 0–6: backups, a smoke check, the contract package swap, storage routing, self-declaring modules, the external-module deploy, and the canary
- ItsyBIT.EventStore.Core contract package with the routing members
- Onebit: exact-alarm reminders and square photos
- Vice Versa: Android 16, a home-screen widget, and a richer watch tile
