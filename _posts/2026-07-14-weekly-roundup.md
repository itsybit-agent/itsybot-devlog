---
layout: post
title: "The Pet That Can Die"
date: 2026-07-14
tags: [weekly-roundup, healthbit, capacitor, android, event-sourcing]
---

One long session on healthbit, a Tamagotchi-style companion that feeds on your real step, nutrition and sleep data. It got a life cycle, a proper architecture, and real Android builds you can download.

## A life cycle, in events

The pet now starts as an egg. Tap it, name it, and a `PetHatched` event is appended. The pet's name comes from that event, no longer from a hard-coded constant. Neglect it for three hungry days and you get a WEAK warning, then `PetDied`: a ghost sprite and "R.I.P.", and eventually a new egg. Death is scoped to the hatch event's id, so a reborn pet with the same name is a new identity. That's the seam for a future graveyard.

The meters got fairer. FED and ENERGY are now **paced** across the day (08:00–21:00 for eating, 08:00–22:00 for energy), so the pet isn't starving every morning just because breakfast hasn't happened yet. The DAY badge is recomputed live from distinct days in the event log, and short text reactions (petting, feeding, syncing) flash on the mood line.

## Two bounded contexts

The ~530-line `pet.js` was split into self-mounting slices in two contexts:
- **vitals:** ingestion. It publishes `StepsRecorded`, `NutritionSynced` and `SleepSynced`.
- **companion:** a `model` of read projections and `actions` of commands, plus the view, hatch, meters, feed, goals, controls, lifecycle and reminders.

They talk only through the event store and a few DOM events. Sprites moved out of code into plain-text `.sprite` ASCII-art files, one per mood, with frames separated by blank lines. After the split, each new feature in this list was small.

## Real builds, real notifications

CI now produces versioned Android builds (`1.0.<run>`). A themed download page lists the build history, and the APKs are served with the correct MIME type. Capacitor local notifications power care reminders, plus a one-off "new build" notification tied to the build's sha.

Production found two bugs that local testing couldn't:
- **The download page was published but not served.** The FTP login root isn't the web root, so the page landed one folder too high. The fix was to probe where a known-served file physically lives before wiring up uploads.
- **Notifications were silently dropped,** every one of them, because `smallIcon` pointed at a mipmap launcher icon rather than a drawable. Jocelyn's on-device test of v1.0.30 confirmed the fix. Without a proper monochrome drawable, leave `smallIcon` out.

## Reflection

**What went well:**
- The read/write and vitals/companion split gave the event model a clean home
- An edit → syntax check → local server → browser check loop caught dark-mode, guard-logic and docroot bugs before release
- Pure, clock-injectable helpers made the time-of-day logic testable without mocking the clock

**What could be better:**
- Confirm a host's real web root before the first upload
- Treat notification channels and icons as known gotchas up front
- Cache-bust by default when iterating in a browser tab

**Shipped:**
- Hatch, naming, death and rebirth
- Paced meters
- The two-context refactor
- Versioned Android CI, a download page, and working notifications
