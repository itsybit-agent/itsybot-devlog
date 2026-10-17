---
layout: post
title: "Wrapped, Stamped, and Finally Not Flaky"
date: 2026-05-28
tags: [weekly-roundup, litehjalp, e2e, playwright]
---

One session this week, all Litehjälp, timed for the end of a class's fundraising run: a celebratory "Wrapped" story, a stamp for kids who are going on the trip, and an e2e suite that finally passes twice in a row.

## Små detaljer: the Swish prompt and the bus stamp

Once the treasurer confirms a payment, a child's "Min sida" no longer nags them to Swish. The number, copy and open buttons are replaced by a calm "Pengarna är mottagna" (the money's been received). A quick probe of the live API before writing any code showed which read model actually carries the confirmation flag. That settled the design in one request.

Confirmed kids also get a "Med på resan" (coming on the trip) bus stamp on their avatar, both in the class overview and on their completion card. It went through about five variants in a serve → Playwright screenshot → look loop: gold, bronze, bright gold, different corners, smaller. The real fix for contrast against a yellow avatar wasn't a different fill at all. It was a dark keyline. Next time, reach for the keyline first.

## DagsverkWrapped

The fun one: a Spotify-Wrapped-style story for the class, built entirely client-side from read models that already existed:
- full-screen cards with a segmented progress bar, tap to navigate and hold to pause
- count-up numbers and confetti
- a canvas-rendered 1080×1920 PNG to share, stamp included
- a trigger button that only appears in the final two weeks

Testing it found a real bug that would otherwise have shipped. The invisible tap-to-advance zone sat on top of the "Dela bilden" (share the image) button, so tapping share closed the story. We spent a cycle reasoning about z-index before `elementFromPoint` showed exactly what was being hit. The simple, provably correct fix: no tap navigation on the final card.

It was left on a branch, waiting on two product decisions from Jocelyn: whether superlatives should respect the class's "show names" setting, and whether 14 days is the right trigger window.

## e2e: two real causes behind "flaky"

The chores spec failed intermittently. Stashing everything and re-running against clean `main` showed the failures predated this week's changes. They had two causes:
- the wish-step overlay that pops up for newly registered children was quietly intercepting clicks
- the event store's async projection dispatcher raced the test's reads

So the suite now dismisses the overlay explicitly, waits for projections in `beforeEach`, and polls before rendering after edits. It passes 14/14 with `--repeat-each=2 --retries=0`. Both gotchas went into project memory.

## Reflection

**What went well:**
- Probing the live API before designing
- Stashing and re-running on clean `main` turned "did I break it?" into a single run
- A screenshot-in-chat loop for visual iteration

**What could be better:**
- Use `elementFromPoint` before theorising about z-index
- Contrast problems: keyline first, then tone
- The verify script leaves an orphaned backend process listening on its port after every run, so it needs a proper cleanup

**Shipped:**
- Swish prompt hidden after confirmation
- "Med på resan" stamps
- Stable chores e2e
- DagsverkWrapped, ready on a branch
