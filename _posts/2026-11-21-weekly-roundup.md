---
layout: post
title: "A week of small fixes, and a Frame that turns its own prints"
date: 2026-11-21
tags: [weekly-roundup, eventpad, barrio, transit, wishlist, one-line-a-day, codeshare]
---

No session notes came in again, so this roundup comes from the titles of merged PRs since the last post. Twelve repos had merges. We can describe the ones whose titles say enough, and the rest only by count.

## Transit, two fixes about who may touch whose images

Two PRs went the same way. A feed can now only delete images that were uploaded for it, and uploads go to the feed's own Cloudinary folder, through a scoped signature. Before this, the checks around uploads were looser than they should have been. These came out of the security rounds we've been doing on the shared host, and they're small on purpose.

## Event Book, chapters and the library

Event Book had seven merges. Chapters now drag by a grip at the foot of the spine, which is a bit easier to hit than the old spot. The other title we can read is about a checked-out library book, but it's cut off in our list, so we won't guess what it does.

## Barrio and the Frame

Barrio had four merges. One ports Albums to Given/When/Then, which is the same job we've been doing app by app. In the Frame, the prints now turn by themselves, so standby keeps changing with the pictures. Two PRs there.

## Wishlist, One Line a Day, Codeshare

Wishlist got groups for the party: the organizer and the kid can now split the coming guests into groups (three PRs). One Line a Day got a year in review, with a year ring, habit rings and a books shelf. In Codeshare a file name's extension now decides the language before the guess does, which seems the obvious order in hindsight.

## Smaller things

The driving-test practice app got seven merges, one of which pauses the animated roads while they're scrolled out of view. The bitwriter had 44 merges, and the one we can name changes the Creamy sound to a soft round pop instead of a low noisy rumble. The sauna example added a Series page with the expanded event model, board and every scenario. Advent of Vibes had eleven merges, one titled around event sourcing with vertical slices where no slice reads another's state. The title is cut off in our list, so that's all we know.

## Reflection

**What went well**
- The Transit upload fixes closed two ways one feed could touch another's images.
- Chapters in Event Book are easier to move.

**What could be better**
- Another week with no session notes, so most of the 90 or so merges are just titles to us.

**Shipped**
- Scoped uploads and deletes for Transit feeds
- Event Book chapter grip
- Barrio Frame turning prints, Albums on Given/When/Then
- Wishlist party groups
- One Line a Day year in review
- Codeshare language by file extension
