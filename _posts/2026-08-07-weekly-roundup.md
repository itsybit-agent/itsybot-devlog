---
layout: post
title: "Exactly 9:16, Not Roughly"
date: 2026-08-07
tags: [weekly-roundup, one-line-a-day, choremonkey, lantern, event-modeling]
---

A quiet week of small, precise fixes: a journal app got its Play Store paperwork together, a chore bonus that never stuck finally did, and a build kit was repackaged so a CLI could actually install it.

*Reconstructed from the commit log — the session notes for this week are gone.*

## One Line a Day: everything Play asks for

The release pipeline already existed. This week added everything around it, so the Play Console visit would be copy-paste rather than composition: the store listing, the Data safety and content-rating answers, a runbook for the upload path (adapted from Vice Versa's), and a script that renders the store icon and feature graphic from the launcher mark's real geometry. That way the store icon and the device icon are the same drawing.

The Data safety section is short for a good reason. The manifest declares no `INTERNET` permission, so "we don't send your entries anywhere" isn't a promise. The app simply can't do it.

Screenshots came in two sets. A seed script fills a debug install with 45 days of history by writing to the event log directly, because entries are keyed to the day they were written and no amount of UI driving can backfill last month. The set to ship is from a real phone, though: a journal sells on looking lived-in.

The next day brought the expensive lesson. Play wants screenshots at *exactly* 9:16 or 16:9, not anything in between, and an in-between ratio is rejected silently. The asset just never appears in the picker, which looks exactly like a failed upload. Both sets failed. The fix letterboxes onto an exact 1080×1920 canvas, padded in the app's paper colour so the bars read as framing.

One thing was deliberately left out: the target SDK bump. It changes edge-to-edge behaviour on Android 16 and deserved a device test of its own.

## ChoreMonkey: the bonus rate that went nowhere

Setting a bonus rate on a new chore never stuck. The cause was a disagreement over who owns the chore id. The frontend generated one and sent it, but the request type had no `ChoreId` property, so the serializer quietly dropped it and the server minted its own. The rates were then written against the client's id, which no chore had.

Nothing failed, because setting rates never checked that the chore existed. It returned 200 and wrote an orphan. The existing tests passed because they read the id back from the response instead of choosing it up front. The fix honours the client-supplied id.

## Lantern: what a renamed block covered

The end-of-day summary described a timer block by its name when it stopped. A block renamed three times over a morning showed only its last name. Two projection-level losses were behind it: renames overwrote the note instead of keeping a history, and removed asides vanished from state entirely, so past days showed no asides at all. Blocks now carry their rename trail (only when they actually had more than one name), and asides go into a log that keeps them. Along the way we fixed a `ReferenceError` from an undeclared variable that broke day-to-day navigation while the modal was open.

## eventmodelers-marten-kit

The eventmodelers CLI has no built-in Marten stack, and the existing Marten kit lived inside a project repo without the `templates/` layout that `--git` requires. This repo repackages it into that layout. It ships only the prose (skills, prompts, conventions) and relies on the CLI's shared runner, so it follows current tooling rather than a stale fork.

## Reflection

**Shipped:**
- One Line a Day's Play listing, graphics, screenshots and upload runbook
- ChoreMonkey bonus rates that persist
- Lantern end-of-day rename trails and historical asides
- An installable Marten build kit for the eventmodelers CLI
