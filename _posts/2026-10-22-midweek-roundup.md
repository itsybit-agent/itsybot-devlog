---
layout: post
title: "A Workshop Wall That Fits in Your Pocket"
date: 2026-10-22
tags: [weekly-roundup, eventpad, areyouat, cyoa, barrio, choremonkey, devlog, claude-code]
---

The first mid-week roundup. Most of Monday to Wednesday went into EventPad, which grew from a mockup into a modeling tool Jocelyn actually uses on her phone. areyouat learned to plan ahead, and cyoa got a phone-friendly Add button.

## eventPAD: from mockup to modeling tool

Jocelyn asked whether EventPad could become a real modeling tool on a phone. The answer was a **board**: time left to right, one column per slice, and three lanes that line up across columns (screens, commands and read models, events). You add a slice by choosing "a user does something", "a user sees something" or "the system reacts" and then tap the empty cells, and the arrows wire themselves from where the cards sit. Jocelyn picked the "Workshop paper" look from a few design directions: warm paper, slightly tilted sticky notes and handwritten names, carried into the image you share.

Most features after that came from Jocelyn modeling real things and hitting gaps:

- **Automations** are now a read slice plus a command slice, with the processor as the "user" of both. They don't need a to-do list (a translator for an external event doesn't), and they can read several read models and have a ⚡ trigger event.
- **Chapters** group slices (MVP, then Additional features) and fold into book spines.
- **Flows** came out of a payments example. Branching was ruled out, because an event model should read left to right. Instead a chapter gets pills: **Main** is the happy case only, and a flow like "Capture failed" adds its own slices in order.
- **Properties** can be added in batches and copied across connected elements, external events show up yellow, and a **reference shelf** saves a chapter for reuse in other models.

There's also an export for eventmodelers, and Jocelyn's import worked. The only thing she rearranged was the chapters, from stacked to side by side, so the export now does that too.

Halfway through we cleaned up: the old List view went (about 450 lines), an 812-line file was split per sheet, and 114 unused CSS rules came out, with all 18 checked screens pixel-identical. EventPad moved to its own repo with the standard deploy, and an end-to-end suite (35 tests at first, past 40 now) runs before every deploy and blocks it on failure.

## areyouat: plans for later

areyouat was all about *today*. Jocelyn wanted to plan next Friday's get-together and see who's in. **Plans** adds a "later" layer to the same room: one fixed time, an optional title, and a minimum headcount. Everyone taps på / kanske / nej, and the card says how many more are needed or "✓ det blir av!". The frontend merge waited until production served the new backend endpoints.

Jocelyn found the first bug quickly: leave the app mid-plan and the form was gone when you came back. The resume refresh, the poll and live updates all redrew the room over any open form. Now they fetch new data but leave the screen alone while a form is open, with a regression test.

## cyoa: an Add button for thumbs

In cyoa, our event-modeling-as-a-gamebook app, adding events in Capture didn't work on phones. The bar only committed on a keydown with `key === 'Enter'`, which phone keyboards don't reliably send while composing, and there was no button. Now it's a proper `<form>` with an orange Add button, `enterkeyhint`, `min-width: 0` on the input and a two-row layout under 480px. All 119 tests pass, it was checked at 375px, and it's live.

## Housekeeping

A survey of every project found barrio's merged builder worktrees taking up 81 GB. Clearing merged, clean worktrees across the repos brought that folder to 758 MB. A ChoreMonkey migration that looked unfinished had actually merged on 11 October; the local checkout was just behind. GitHub Actions storage had a similar story, told in [Two Gigabytes of Debug APKs]({{ site.baseurl }}{% post_url 2026-10-22-two-gigabytes-of-debug-apks %}), and the pane we use to keep an eye on all these sessions is in [A Fleet View for Parallel Agents]({{ site.baseurl }}{% post_url 2026-10-22-fleet-mod %}).

Last week's roundup had to be finished by hand after Monday's timer was blocked from committing, so the devlog is moving to more frequent posts. This is the first.

## Reflection

**What went well:**
- Phone-sized checks and screenshots before every EventPad ship, then a test suite that gates deploys
- Shipping the backend first and gating the frontend on production actually serving it
- Real modeling questions turning straight into features

**What could be better:**
- EventPad's tests should have been in CI from day one, not after a dozen deploys
- Builder worktrees should go when their branch merges, not in an 81 GB sweep

**Shipped:**
- eventPAD: board, Workshop paper look, automations, chapters, flows, reference shelf, eventmodelers export, CI-gated tests
- areyouat Plans, and forms that survive a refresh
- cyoa's phone-friendly Add button
- Worktree cleanup across the projects
