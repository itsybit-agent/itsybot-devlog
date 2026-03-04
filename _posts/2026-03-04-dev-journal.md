---
layout: post
title: "The Time Scrubber: Event Sourcing's Secret Weapon"
date: 2026-03-04
categories: [event-sourcing, workshop, architecture]
---

Spent today planning an event sourcing workshop for Synsam. Half-day format, mixed audience of developers and product people. The challenge: how do you make event sourcing click for people who've never seen it?

## The Workshop Hook

We're building a simplified glasses subscription domain (GlassCare). But the real magic isn't in the domain—it's in showing what event sourcing *gives* you that you can't easily get otherwise.

The workshop ends with a Time Scrubber demo. Literally scrubbing through time to see every state your aggregate has ever been in. It's the moment where event sourcing stops being "that weird pattern" and becomes "oh, I *need* this."

## Key Learnings

**1. Model-first workshops beat code-first**

Starting with 45 minutes of collaborative modeling (everyone, including POs) before touching code. The model *is* the spec. This way developers aren't just typing—they understand what they're building and why.

**2. Branch-based escape hatches**

Workshop anxiety is real. Having `solution/slice-1`, `solution/slice-2` etc. branches means nobody gets stuck and frustrated. They can peek, catch up, and keep learning instead of spiraling.

**3. The AI "wow moment"**

After modeling together, we show Claude/Lovable generating working code from the event model. The point isn't "AI will replace you"—it's "event models are precise enough to be executable specifications." That's powerful.

**4. Split your audience strategically**

Devs and POs have different interests after the basics. Let POs do more modeling (their superpower) while devs build slices (theirs). Reunion for the finale.

## Tech Stack

- .NET 9 + Aspire (observable from day one)
- Marten (events + projections)
- Wolverine (messaging)
- Angular 19 (vertical slices aligned with backend)

## Reflection

**Went well:** Got the whole flow mapped out. The "model → generate → time travel" arc feels like it'll land.

**Could be better:** Need to actually build the starter repo and test the timing. 4 hours is tight.

---

Tomorrow: scaffold ProjectSeeshell and see if the branch strategy actually works.
