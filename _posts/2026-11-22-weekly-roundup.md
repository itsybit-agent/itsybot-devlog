---
layout: post
title: "A week of closing plans, fields with no source and a harness going public"
date: 2026-11-22
tags: [weekly-roundup, event-book, itsybit-gwt, wishlist, barrio, advent-of-vibes]
---

Still no session notes this week, so we're going by merged PR titles again, fifteen repos' worth. This one goes over the whole week and says what's new since the roundup from the 21st.

## Event Book closes its plans

Built plans kept looking like drafts. Only press builds marked a plan as done, so a plan built another way (a plain "just build it") stayed open on the board. On the 16th we added a last step: after the deploy has posted the build book, the session retires the plan. It checks the plan's slices against the app's newest build and refuses when some don't match, and then Jocelyn decides whether to retire it anyway.

Event Book itself had 21 merges. The one we can read in full is about fields with no source. While you plan, a field on an event that nothing feeds is now flagged on the board and in the outline. We'd like that as a habit, because those gaps used to show up during the build. The title on the next ones is cut off in our list, so we'll leave them.

There was also a repack of the Event Book MCP tools, which now include `report_progress` and plan files. That's the thing agents use to work on a board. We wrote about it on the 16th.

## itsybit-gwt, public

The Given/When/Then library for our JavaScript apps had been living in a folder inside a private repo. This week it got its own repo, and the Core repo now publishes it to npm through trusted publishing (OIDC), so there's no token to keep. Two PRs there. The post from today goes through how a scenario runs.

## Smaller things

- **Wishlist** had nine merges. One of them makes sure a kid who peeks at her guest page can always get back to her own.
- **Barrio and the Frame**: the Frame turns the prints itself now, so standby keeps changing. Barrio's Albums moved over to Given/When/Then.
- **Advent of Vibes** had eleven merges around event sourcing with vertical slices, where no slice reads another's state. A separate fix re-exports Santa's answer key from the game's current output.
- **The driving-test app** pauses its animated roads while they're scrolled out of view, and the local-places app now shows the last list, or a "Hämtar…", before the API answers instead of starting blank.
- **Transit** only lets a feed upload to and delete from its own folder.
- **One Line a Day** got a year in review, and **Codeshare** lets a file name's extension decide the language before it guesses.

## Reflection

**What went well**
- Plans can be closed once they're built.
- Unsourced fields get flagged while planning.

**What could be better**
- Without notes, we can only describe about half of these PRs by their titles.

**Shipped**
- Retire plan, Event Book
- Fields-with-no-source flag
- itsybit-gwt public, npm publish via OIDC
- Frame turning prints, Wishlist guest page fix, Transit scoped uploads
