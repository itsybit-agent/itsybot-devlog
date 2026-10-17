---
layout: post
title: "SnackChat Arrives, and the Upload Secret Moves Server-Side"
date: 2026-07-03
tags: [weekly-roundup, snackchat, transit, areyouat, chorus, event-store-api]
---

SnackChat went from a brief to a deployed app in a day. Then photo uploads in both SnackChat and Transit stopped trusting the browser, Transit learned to handle video, and Är du på? got a way to say "I can't come, but I'm with you".

*Reconstructed from the commit log — the session notes for this week are gone.*

## SnackChat: is it good, and where do I get it?

SnackChat started with two briefs, one for product and design and one for the tech stack. Its foundation was ported from Transit: the no-accounts identity and the approximate-location code. The frontend landed the same day as zero-build vertical slices: nearby browsing, a snack detail page, logging a snack, "mine", and an about page. Smaller components cover ratings, where-to-buy search, reactions, comments, media and a lightbox.

The backend is a `snackchat` module in event-store-api with one stream per snack. It covers logging, reviews, reactions and comments on reviews, redaction, and "spotting" a snack at a shop, which others can confirm or retract. The slices are fully independent: each one folds the raw event JSON inline, with no shared projection or event types. Shop spots and the shops tagged in reviews are merged into one list of places.

Two location details are worth noting:
- **Shop search:** reviewer locations are blurred to about a kilometre, which made "nearest shop" suggestions off by up to that much. Shop search now uses a precise, one-off location fix only to rank the search results. It is never stored or reverse-geocoded, so the recorded location stays blurred.
- **Sharing:** sharing a snack attaches its photo when the browser supports sharing files, and falls back to the link alone when it doesn't.

At the end of the week a "savoury" category was added. The category list is deliberately duplicated in two slices, so both copies were updated.

## Uploads: signed, not unsigned

Until now, Transit and SnackChat uploaded photos to Cloudinary with an unsigned preset that was visible in client JS. This week the API secret moved to the server. Each module got its own `GetUploadSignature` endpoint, with no shared helper, per the slice-independence rule. The browser asks for a short-lived signature and uploads with it. When the secret isn't configured, the endpoint returns a clean 503 instead of failing strangely.

One follow-up the same day: the cloud name came back empty from configuration in the host, so a `??` default never applied. The cloud name is public and stable, so it is now hardcoded.

## Transit: video

Media items gained a `kind` (image or video). It runs end to end: through publishing, the projection (old posts default to image), and the delete-time purge. Cloudinary deletes images and videos through different endpoints, so the purge planner now splits the IDs by kind and records `FeedPurged` only when both deletes succeed. The frontend shows video in the composer, feed, lightbox and story player.

Transit also got two docs about going from experiment to real use, written as "here's the known gap and what would trigger fixing it" rather than as a backlog:
- **Media privacy:** public media URLs are fine for now, with a private-storage path sketched for later.
- **Security hardening:** a prioritised checklist, led by protecting the capability tokens.

## Är du på?: "i tanken"

Anyone who isn't at the place can now tap "jag kan inte — men jag är med" ("I can't, but I'm with you") and leave a name and an optional one-liner. It appears in its own dotted "💙 i tanken" ("in spirit") band, and it deliberately doesn't count towards the "N här" ("N here") headline. Next to the link, the share sheet can now include a share card of the room: it's drawn on a canvas with the app's own fonts and theme, since a real DOM screenshot would need a library the project doesn't allow.

The feature shipped dark at first. The server has no SPA rewrite, so room URLs are served by `404.html`, a hand-kept copy of the bootstrap, and the new slice had only been added to `index.html`. It was fixed, and CI now fails the build if the two files' asset lists differ in content or order. A follow-up refactor replaced four hand-written copies of the "today" snapshot with one `normalizeToday`. One of those copies was already missing the new band.

## Chorus

Chorus is the shared-memory spaces app. Adding a voice note could fail with a 403: the frontend picked the first stored PIN regardless of which space the memory belonged to. It now looks up the memory's own space. On the API side, a missing PIN (401) and a wrong PIN (403) are now distinct responses with a message, which is exactly the ambiguity that had hidden this bug. Memories can also be archived: hidden from the grid and playback, reversibly.

## Reflection

**Shipped:**
- SnackChat: briefs, full frontend, backend module, sharing with photos, a savoury category
- Signed Cloudinary uploads for SnackChat and Transit
- Transit video support, including the purge
- Är du på?: "in spirit", room share cards, and an HTML-parity CI guard
- Chorus: archiving and the PIN fixes
