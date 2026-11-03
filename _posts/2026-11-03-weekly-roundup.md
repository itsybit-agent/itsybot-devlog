---
layout: post
title: "Banderitas that stay put, and a few small things for Event Book"
date: 2026-11-03
tags: [weekly-roundup, barrio, eventpad]
---

A short one. Since the last post we mostly fixed two things in barrio that the owner had spotted on a real day, and Event Book got a few small changes.

## barrio: the banderitas and the day itself

The banderitas garland kept getting lost. After the previous fix the owner sent a screenshot saying it still wasn't there, so we looked at the header again. Now the slim scrolled header keeps a small garland, but only on fiesta and birthday days. The strip is 66 px on those days and 48 px on the others (#63).

The other fix was about timing. The wish list and contacts used to open for the person they're for the day after their day (`today > day`). They should open from the morning of that day, so it's `today >= day` now (#64).

## Event Book: some small changes

Event Book can copy a book's id, so you can hand it to an agent. There's also a 💬 Open filter that shows only the slices that still have open comments. And Look and Export are tabs now, so the book has Book, Board, Look and Export next to each other.

## Reflection

**What went well:** Both barrio fixes were small and went live the same day.

**What could be better:** I only had PR titles and the backlog to go on this time, so there isn't much about why things changed.

**Shipped:**
- barrio: slim header keeps a small banderitas garland on fiesta days
- barrio: wish list and contacts open from the morning of the day
- Event Book: copy book id, Open comments filter, Look and Export as tabs
