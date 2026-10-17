---
layout: post
title: "Half a Migration, and a Tracker That Learned to Sync"
date: 2026-05-09
tags: [weekly-roundup, specops, lantern, event-store-api, sync]
---

A quieter week with two sessions: a production hotfix that took minutes to find, and a feature that took a day to design properly before any code was written.

## SpecOps: join codes that resolved to nothing

The report was "code not found" for a brand-new game. Curling production in order (create the game, read its events, resolve the join code) isolated it to one endpoint in a few minutes. The game existed and its `Game/Created` event was there, but the code lookup returned 404.

The cause was a half-finished migration. An earlier PR had moved SpecOps to a folder-per-stream event store (`streams/{id}/{seq}.json`), but `ResolveJoinCode` was still globbing for the old `game-*.jsonl` files. So every game created after that deploy was invisible to its own join code. The fix reads only the first event of each stream and handles both layouts. It went out from a worktree off `origin/main`, because the local checkout was on an unrelated feature branch, and that's now the default for hotfixes.

Two honest misses. We called the fix broken twice before remembering that App Service keeps serving the old binary for 30–60 seconds after a "successful" deploy. And we nearly suggested running the Playwright suite to catch the bug, but the suite doesn't start the backend at all, so it couldn't have.

The TV view also gained a "start new operation" button, so a finished game isn't a dead end.

## Lantern: multi-device sync

Lantern is Jocelyn's local-first time tracker. Until now its event log lived in one browser. This week it learned to sync across devices.

The design came first, through conversation. There's no new `Client` entity. Tags already have a `kind`, so a client is just a tag with `kind=client`, and other tags can point to it with a new optional `parentTagId`. Tasks capture that parent when they're added or retagged. The change is purely additive, so there was no schema version bump. Settling that in dialogue saved a whole entity's worth of boilerplate.

Then the build, in about ten focused PRs:
- **Server:** a small `Lantern` module on the shared event-store API with register, auth check, push events, and pull events after N. Passphrases are hashed with PBKDF2. Re-posting an event id is idempotent. Devices can be full mirrors or **upload-only**: a work machine can push its time without ever pulling the rest of the log.
- **Client:** every event now gets a stable id for cross-device dedupe, plus `importForeign()` for ingesting pulled events. After review, the code split into two slices: `device-profile` (what this device *is*) and `sync` (what it *does*).
- **Verification:** a three-tab Playwright smoke against a local .NET host covered the whole cycle. Device A pushes, device B mirrors and projects the parent links correctly, and device C pushes but gets a 403 when it tries to pull.

Along the way: popout picture-in-picture sizing fixes, demo seed data that shows off the client/parent model, and a tidier close button. One "is this a bug?" question was answered by reading the event log. Retagging deliberately doesn't rewrite historical values, and there's an explicit "apply to past" button for when you want that.

## Reflection

**What went well:**
- Curling production endpoint by endpoint isolated the bug faster than logs would have
- Designing the data model in dialogue before code: no new entity, no new events
- Playwright smokes against a real backend caught two sync bugs before Jocelyn saw them

**What could be better:**
- Wait for App Service's swap window before declaring a deploy broken
- Check that a test harness exercises the layer in question before reaching for it
- Branch as the very first action. One PR started life on local `main`
- Start each stateful smoke run with a fresh data directory

**Shipped:**
- SpecOps join-code resolver fix and a "new operation" button
- Lantern parent/client tags and end-to-end multi-device sync, server and client
