---
layout: post
title: "Close the Books, Count the Books"
date: 2026-08-16
tags: [weekly-roundup, viceversa, onebit, chorus, billsbillsbills, itsybit-se, android, event-sourcing]
---

Two Android apps had a busy week. Vice Versa learned to undo, reflect and close a day. One Line a Day learned to tell books apart and came out the other side renamed Onebit.

*Reconstructed from the commit log — the session notes for this week are gone.*

## Vice Versa: undo, "worth it?", and closing the books

The week opened with a toolchain jump: AGP 9, Kotlin 2.4, compile SDK 37. API 37 only ships in a new platform layout that AGP 8 can't read. The watch stays on target 36, because nothing in the field runs 37 and we couldn't test it.

Then the features, each one an event like everything else:
- **Night mode,** a warm dark "ink" palette rather than grey. The theme choice is a `ThemeChosen` event, so it rides along in backup and restore with no separate storage.
- **Undo an indulge.** Tasks always had an inverse, but vices didn't. That was survivable when credit reset at midnight. Once the pool started carrying over, a mis-tap cost credit permanently, which is a poor fit for an app whose first principle is "help, not shame". `ViceUnredeemed` mirrors `TaskUncompleted`.
- **"Worth it?"** A single question asked once an indulgence has settled. Over time a vice carries a plain line like "worth it 1 of the last 3". It gates nothing, and both answers are styled identically. The moment "Worth it" looks like the right answer, it stops being a mirror and becomes a scold.
- **Starters instead of an empty screen.** Tapping a suggestion emits an ordinary `TaskDefined` or `ViceDefined`, so the first tap teaches the mechanic.
- **Close the books on a day.** `DayClosed` is written automatically the first time the app notices a later day has begun. Missing ten days closes ten days, quietly, in order, and empty days are skipped. "Look back" in Settings is the ledger.

Two real bugs came out of it. An event type the build couldn't decode threw inside the event-log flow and blanked the whole app. That's reachable by restoring a newer backup or rolling back a version, and unknown events are now skipped. Separately, the home screen captured "now" once at startup and never updated it, so an evening vice never unlocked while the app was open and midnight never rolled over.

On the release side, the Wear OS build is now automated as its own Play edit. Putting both bundles in one edit is what had failed earlier, with an opaque internal error. Production runs can name their own version, and release notes are checked against Play's 500-character cap before the upload rather than after. There was also a typography sweep: a bare `Text()` resolves to the serif body style, so 31 controls had been rendering in the voice font.

## Onebit: one book, however you spell it

The journal's reading habit had a data problem. The same book typed slightly differently on different days would count as several books. Reading habits now offer titles from Open Library (no key, no account) and record the work id next to the note, which stays exactly what a person wrote. Your own recent notes come first, straight from the log, with no network involved. Finishing a book became its own event, so a later note can't erase it, and habits now declare their kind instead of having it guessed from their name.

Also this week: an optional app lock using the phone's own biometric or screen lock, with a fix for a prompt that only worked every other time. There's a dark palette too. Backups are now actually stamped with their format version, which had been silently omitted. The release notes were corrected so they no longer promise "no network" now that book lookup exists. Shareable day and stats cards arrived, along with a readable event log you can export as `.txt`. The app was then renamed **Onebit**, and the backup tag moved with it, keeping old exports readable.

## Chorus: a cold start that looked like data loss

A reported "error fetching memories" eventually reproduced. Every request failed at the same instant and then succeeded: an API cold start. Reads now retry with backoff. Only GET and HEAD retry, since a failed write may already have landed. The player no longer says "No memories yet" when loads *failed*. Two other defects turned up on the same path: a client call to a route that only accepts POST, and a stale PIN that bricked the player instead of falling back to the public view. The app was also repainted from cyan to a light "Paper" palette with a warm dark "Ember" for fullscreen, with every accent derived from one `--accent` token.

## Smaller things

- **BillsBillsBills:** skip rules only ran at import time, so rows imported before a rule existed stayed forever. Re-importing now appends compensating deletes for them.
- **itsybit.se:** the projects catalog gained Vice Versa and the Onebit rename, and cards can show a Google Play link.

## Reflection

**Shipped:**
- Vice Versa: night mode, undo, "worth it?", starters, day closing, an automated Wear release
- Onebit: book lookup, finishes, app lock, dark mode, share cards, the rename
- Chorus resilience and the Paper/Ember palette
- A BillsBillsBills import clean-up
- The catalog refresh on itsybit.se
