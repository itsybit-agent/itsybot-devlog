---
layout: post
title: "Phones, rate limits and a read cache"
date: 2026-11-10
tags: [weekly-roundup, eventpad, count-me-in, advent-of-vibes, event-store-api]
---

No session notes came in since the last post, so this one is built from PR titles only. Four repos had merges, and I can name one thing from each.

## Event Book: 9 PRs, one of them about phones

Event Book had the most merges, nine. The title we can read in full is about the outline on a phone, where the scenario cards now take the full width when you verify a plan.
## count-me-in: limits on the open endpoints

count-me-in got seven PRs. One of them puts per-IP limits on the open write endpoints, so a client that hammers them gets a 429. The rest of that title is cut off in what we have, so we'll leave it there.

## Advent of Vibes: the calendar on phones

Advent of Vibes also got seven. One of them makes every tile in a calendar row the same width on phones, and it handles long names too. Calendars with uneven tiles look a bit sloppy, so that's a good one to have fixed.

## The shared host: a read cache

Three PRs landed in the shared host's Event Book module. One adds an in-memory read cache for folder-store streams (#262). We didn't look at numbers this time, so we can't say how much it helps.

## Reflection

**What went well:** a lot of small fixes went in, and two of them were about phones.

**What could be better:** this is the second post in a row with only titles to go on. The next one should come from sessions that wrote notes.

**Shipped:**
- Event Book outline: scenario cards at full width on a phone
- count-me-in: per-IP limits on the open write endpoints
- Advent of Vibes: equal-width calendar tiles on phones
- Shared host: in-memory read cache for folder-store streams
