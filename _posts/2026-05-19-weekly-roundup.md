---
layout: post
title: "Planned Time, Planned Money, and No Double-Counting"
date: 2026-05-19
tags: [weekly-roundup, lantern, litehjalp, event-store-api]
---

A short week with two threads. In Lantern, planned time learned to become recorded time without counting it twice. In Litehjälp, the class overview started showing money that's promised but not yet in.

*Reconstructed from the commit log — the session notes for this week are gone.*

## Lantern: confirming what was planned

Lantern is a tag-based time tracker: vanilla JS, with IndexedDB as an append-only event store and every view a fold over the log. A planned block, like a meeting in the calendar, used to stay "planned" forever. Now, once its end time has passed on today's view, it gets a tinted fill in the strip and a ✓ button in the entries list. The ✓ replaces ▶ Start on those rows, because "start this now" makes no sense for a meeting that already happened. Clicking it emits a new `TaskConfirmed` event, which turns the block into a recorded block at its planned times. It also snapshots the tag's billing settings at confirm time, the same rule recorded blocks follow.

That opened a bug straight away. Before confirming, a planned block that overlapped a recorded block counted for nothing, because planned time gives way to recorded time. After confirming, both blocks were recorded, and the aggregator deliberately doesn't reconcile two recorded blocks against each other, so the overlap counted twice. So confirming now checks for overlaps first and offers to trim:
- **Fully covered:** the recorded block is deleted.
- **Overlap at one edge:** it's rescheduled to drop that edge.
- **Planned block in the middle:** it's split into two halves.

A dialog previews every edit, and the trims are emitted as their own events before `TaskConfirmed`, so the log reads cause → effect.

Overlaps between recorded blocks are also now *visible*. The day total stays as it was, but the overlapping rows get a red dashed bar, a "double-counted" notice lists each conflicting pair, and the strip shows a hatched overlay. Lantern doesn't pick a winner. Only the person tracking knows which block was the real one.

Then came the clean-up. An audit after those PRs moved the confirm-and-trim logic out of the entries list into a shared module and made the strip use the aggregator's overlap list instead of its own scan. It also renamed `read-model.js` to `state.js`, because it's one fold into one app-state object and "state" says so. None of this changed behaviour.

## Litehjälp: the planned slice of the road

Litehjälp's class overview shows fundraising progress as a journey road. That view already showed each child's planned chores. Now the class total does too. In `event-store-api`, the class-progress projection tracks outstanding planned amounts per chore. A planned amount moves into the realised total when the chore is completed and drops out when it's removed. Older stored state gets re-folded on its first read. The web side draws that planned amount as a hatched overlay just past the realised fill, and shows nothing if the field is missing, so it stayed correct while the backend was still being deployed.

## Reflection

**Shipped:**
- Lantern: confirm elapsed planned blocks, overlap-aware trimming, and visible double-count warnings
- Lantern: a shared confirm module and the `state.js` rename
- Litehjälp: planned-chore totals in the class projection and a hatched overlay on the overview
