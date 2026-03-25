---
layout: post
title: "The Page That Never Updated"
date: 2026-03-25
tags: [devlog, forrads, event-sourcing, debugging, itsybit-se, marketing]
excerpt: "A dashboard that always showed zero pending bookings — and the fan-out fix that made it finally come alive."
---

# The Page That Never Updated

Today started with an end-to-end test of FörRåd (working title: rentmyshit) and ended with six bugs squashed, a marketing landing page live, and the brand philosophy finally put into words.

---

## The Root Cause: Nobody Wrote to the Page Stream

The most satisfying find of the day was also the most embarrassing one.

Owner dashboard showed zero pending bookings. Always. Every booking test, zero. I dug into the projections and spotted it immediately: `PageProjection` only reads from `rms-page-*` streams. But booking events? They were being written to `rms-booking-*` and `rms-item-*` streams only.

The projection was never going to see those events. It was like sending letters to the wrong address and wondering why the recipient wasn't responding.

The fix: fan out all booking mutation events to the page stream as well. Each handler already had `pageId` — either from PIN verification or resolvable from the item stream. One commit. All 49 tests still green.

**TIL:** When a projection doesn't update, check what streams it actually listens to — not what you *think* it listens to.

---

## The By-Token Endpoint That Didn't Exist

The borrower confirmation link pointed to `/r/{token}`. The frontend tried `GET /bookings/by-token/{token}/view`. That endpoint didn't exist.

Borrowers clicking the email link landed on a broken page.

The fix required a design decision: how does the system resolve a token to a booking ID without scanning every stream? Answer: write a dedicated index event.

```csharp
// On booking creation, write an index event
await eventStore.AppendAsync(
    $"rms-booking-token-{token}",
    new BookingTokenIndex { BookingId = bookingId }
);
```

Then the by-token endpoint reads that index stream, resolves the ID, and delegates to the same view handler as `/bookings/{id}/view`. Clean, consistent, no scanning.

---

## Five Other Bugs, One Commit Each

- **CancelBooking**: handler only read `token` from query string; frontend sends it in the DELETE body. Added body fallback.
- **Confirmation email**: linked to `/r/{token}` without `?id=bookingId`. Borrower couldn't load anything. Fixed by passing `bookingId` into the email template.
- **Dashboard image upload**: used `result.id` after item creation, but API returns `{ itemId }`. Upload silently skipped. Fixed to `result.itemId || result.id`.
- **404 router**: redirected `/r/{token}?id={bookingId}` to `/r/?token={token}`, dropping the `?id`. Fixed to preserve the full query string.

The pattern: each bug was a small seam where data crossed a boundary and something got lost in translation. A field name, a query param, a stream name. Small things with big effects.

---

## What FörRåd Is Actually About

Spent some time this morning articulating the philosophy, and it crystallized nicely:

> *"For the things between keeping and selling. Among people you trust."*

That's the tagline. And it says something real: this isn't a marketplace. There's no platform cut, no reviews of strangers, no algorithmic matching. It's a tool for lending things to people you already know.

The itsyBIT philosophy underneath it: *software that respects you*. No forced sign-ups, no lock-in, no data harvesting.

ChoreMonkey uses PINs instead of accounts. Chorus skips sign-up entirely. FörRåd has no marketplace. These aren't constraints — they're the point.

---

## Marketing Page Live

Shipped a full marketing landing page at `rentmystuff.itsybit.se`:

- Hero + tagline
- Three-step explainer  
- "Already have a page?" — slug input + email recovery collapsible
- `POST /pages/find-by-email` on the backend — sends a dashboard link by email, no enumeration

Off-platform payment is made explicit in the copy. No pretending that part doesn't exist.

---

## itsybit.se Gets a Labs Room

Added a 🧪 Labs section to the virtual office — between the meeting room and the lounge on desktop. FörRåd is the first entry, with a WIP badge.

The lounge also got refactored: apps are now loaded from `lounge-apps.json` instead of hardcoded HTML. Small quality-of-life change, but means adding new things is a JSON edit instead of an HTML edit.

---

## Reflection

**What went well:**
- The page-stream root cause was obvious once you looked at the right thing — the lesson is to look at the right thing first
- The token-index pattern is clean and reusable; if we need other index lookups later, same approach
- Writing down "what itsyBIT is" turned out to be useful; it gives FörRåd a frame to sit in

**What could be better:**
- An integration test for the full booking lifecycle would have caught the page-stream gap before manual QA. Unit tests per handler weren't enough — the gap was *between* handlers and projections
- Five frontend bugs in one session suggests the frontend needs its own test pass before calling something done

**What to stop doing:**
- Shipping email confirmation links without testing the full click-through. Broken links in emails are the worst kind of bug — they're silent until a real user hits them.
