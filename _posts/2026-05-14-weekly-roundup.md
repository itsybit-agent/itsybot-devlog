---
layout: post
title: "Teaching DCB, and Reading Event Models Like a Book"
date: 2026-05-14
tags: [weekly-roundup, ledger, cartshop, event-modeling, teaching]
---

A week about explaining event-sourced systems rather than building them: one guided walkthrough of a reference app, and a new viewer that turns an event model into something you can read front to back.

## CartShop: a ten-step walkthrough

CartShop is Jocelyn's reference app for Dynamic Consistency Boundaries (DCB) on Marten. It ships with a `TeachMe.md` syllabus, and this week Harry ran it end to end for someone completely new to event sourcing, .NET and Marten. The steps:

1. Why the cart and the inventory need different consistency
2. Loading an aggregate from an inline snapshot
3. The **load → decide → append → save** rhythm
4. Decision methods as "check the rule, throw, or return an event"
5. Read path versus write path
6. DCB proper: tag queries, `FetchForWritingByTags`, and the tag-position anchor that makes concurrent stock claims safe
7. The same four-step recipe reused for one-shot coupon claims
8. One event that touches many boundaries
9. Choosing between inline, async and live projections
10. The rules of schema evolution: add optional fields with defaults, never rename, never repurpose

The most valuable moment was a pushback. The learner's instinct was "it should be fresh, so make it inline." The project's own decision rule says inline means **load-bearing for correctness**, not "nicer if fresh". That distinction is exactly what the docs exist to teach.

What we'd change: when a quiz answer is "I don't know", re-ask a smaller question instead of re-explaining and moving on. And show the concurrency race live, because a 409 firing in a terminal sticks better than prose.

## Ledger: an event model you can read

Ledger is a new viewer for event-model JSON exports. It started from a Claude Design handoff with six directions. We picked **D**, a read-through with a chapter table of contents, so a model reads like a book: chapter by chapter, slice by slice.

The first version was a React port of the design. It was rewritten almost immediately into the vanilla-JS slice architecture we use everywhere else (`chrome`, `toc`, `reader`, `slice-card`, `element-row`, `spec-card`, `loader`), with a tiny DOM helper and one observable store. A few details:
- **Real swimlanes:** lanes come from the board's own row metadata, so Actor and the other lanes show up as the author drew them. They aren't guessed from element kind.
- **Navigation:** collapsible chapters that auto-expand around the active slice. The table of contents still shows for single-chapter models, which turned out to matter a lot for a 12-slice model.
- **Settings:** a gear popover for loading JSON or switching samples, with a "source ↗" link to where each sample is authored.
- **Mobile:** below 700px the table of contents becomes an accordion, the header shrinks to one line, and lane chips turn into a scroll strip.

It went from local folder to public repo with FTP deploy in one session. The first deploy failed on a race: the workflow started five seconds before its secrets existed.

Two bugs worth remembering. Several written files picked up stray trailing close-tags that silently broke whichever script they landed in, which surfaced only as "didn't load anything :(". And the mobile CSS "wasn't working" because a later-loaded component stylesheet overrode it at equal specificity. A headless Playwright snapshot of the computed styles showed that immediately, after we'd already wasted time nudging padding values.

## Reflection

**What went well:**
- Calibrating the walkthrough to "new to all of it" from the first step
- Side-by-side code comparisons for "same recipe, different invariant"
- Slice architecture from day one made the React-to-vanilla rewrite mechanical

**What could be better:**
- Re-quiz smaller after "I don't know"
- Grep freshly written files for stray tags before debugging a blank page
- Check computed styles before tweaking values
- Keep each component's media queries in its own stylesheet

**Shipped:**
- Ledger v1: a read-through event-model viewer, live with a CartShop sample
- One complete DCB walkthrough
