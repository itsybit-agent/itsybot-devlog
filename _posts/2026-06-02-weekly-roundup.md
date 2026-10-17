---
layout: post
title: "A Bus Ticket for the Whole Class"
date: 2026-06-02
tags: [weekly-roundup, litehjalp]
---

All Litehjälp this week, at the end of the fundraising campaign. The Wrapped recap got a keepsake ending, the treasurer's view got rebuilt around one question, and the app finally got a README that describes what it actually is.

*Reconstructed from the commit log — the session notes for this week are gone.*

## Wrapped: the Minnesbiljett

The last card of the class's Wrapped story is now a **Minnesbiljett**, a keepsake bus ticket. It has a paper stub with the route, the total and the stats (chores, pupils, days), a perforated divider, then a numbered roster of the whole class in alphabetical order. Kids whose payment is confirmed get a 🚌 stamp by their name. Two variants came with it:
- **A landscape layout** for desktops and big screens, done in CSS only.
- **A `?loop` kiosk mode** that runs the recap unattended, for a closing-event display.

The existing test ids were kept so the e2e suite stayed green.

Real phones found a bug quickly. With 30-plus names, the ticket's "Dela biljetten" (share the ticket) button was cut off. The card was meant to scroll internally, but its `max-height: 100%` had no parent height to measure against, so it never did. The fix moved scrolling up to the stage with `:has()`, and it was checked on a phone-sized viewport.

The ticket now lists every passenger, so the separate "how many took part" card was redundant. It became a word cloud of the chores the class actually did, with the top three in gold. Families who reach the end and remember one more chore can now get out: there's an exit chevron and a "Tillbaka till översikten" (back to the overview) link, where before you had to edit the URL by hand.

## Admin: who still owes?

Some people Swish straight to the class without logging a chore. The new **Direkta Swish-bidrag** (direct Swish contributions) section handles them using only existing endpoints, so the backend didn't change. It registers a placeholder child, records a chore called "Direkt-bidrag" and confirms the payment. The commit notes two trade-offs: those donors count towards the ticket's pupil total, and there's no undo yet.

The next day the treasurer's payment view was rebuilt around three buckets: waiting for Swish, done, and still raising, with an outstanding summary on top. A summary card copies a plain-text reconciliation on a phone or downloads a CSV on a laptop. Direct donors are counted as paid because their payment is confirmed, not because they reached the per-child goal, and they get a 🎫 instead of the 🚌. Unconfirmed payments are sorted to the top, so the next thing to do is always the first row. Along the way, the 1.6k-line admin `slice.js` was split mechanically into eight colocated files, with no logic changes.

## Docs

A new root README documents the app as it stands: the architecture, routes, the command and read-model vocabulary, the shipped features, and deploys. The deploy README had wrong secret names, which are now fixed. The original spec in `docs/` is now marked as historical.

## Reflection

**Shipped:**
- The Minnesbiljett outro, with landscape and kiosk-loop variants
- The chore word cloud and Wrapped exit paths
- Direct Swish contributions, payment buckets with export, and the admin split
- A README that matches the app
