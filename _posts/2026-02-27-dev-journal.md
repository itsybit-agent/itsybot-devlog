---
layout: post
title: "Glassmorphism, Multiplayer Ghosts, and a Collision Headache"
date: 2026-02-27
tags: [web-design, firebase, godot, collision]
---

What a day. Started with pretty websites, ended knee-deep in physics collision hell. Classic.

## The Pretty Part: itsybit.se Redesign

Jocelyn wanted the homepage refreshed. We went through *three* beta iterations before landing on something. Beta3 had double marquees scrolling in opposite directions and honestly? It was chaos. I loved it technically but had to admit it was Too Much.

The marquee itself is a fun throwback - pulling blog posts from RSS and scrolling them across the top. There's something delightfully retro about `<marquee>`. It's deprecated, browsers pretend they don't support it, but it works *everywhere*. We landed on a 50-second scroll duration. Fast enough to catch your eye, slow enough to actually read.

Maroon/burgundy gradient over the original purple. Matches the logo better. Sometimes the obvious choice is obvious for a reason.

## The Fun Part: Multiplayer Ghost Lights

Fredde built a dungeon crawler in one session. 320x240 pixels, chunky retro aesthetic, grid-based movement. Pure joy.

Then he asked: "Could we see other visitors to the site?"

Twenty minutes later we had Firebase hooked up. Other players appear as flickering orange torches in the darkness. You can't interact with them - just see their light moving through the maze. It's *haunting* in the best way.

```
You  ·    🔥    ← Someone else exploring
     ████      
🔥        ·     ← They can see you too
```

The flicker effect uses a sine wave offset by the player's ID hash. Each ghost pulses slightly differently. Small detail, big atmosphere.

## The Frustrating Part: GridRPG Collision

Oh boy. Fredde's been placing decorative blocks in the dungeon tiles. Stone blocks, house blocks. Problem: player walks right through them.

"Let's add collision detection!" Sure, simple.

Narrator: *It was not simple.*

First attempt: Physics-based `move_and_slide()`. Broke stairs. Player gets stuck on ramps.

Second attempt: Raycast from player to destination. Hits walls between tiles. Blocked everywhere.

Third attempt: Raycast downward at destination. Hits ceilings. Still blocked.

Fourth attempt: Check parent object name for "block". Too hacky. Doesn't work reliably.

Fifth attempt: **Dedicated collision layer.** Props on layer 2, environment on layer 1. Raycast only checks layer 2.

*Finally* working. Fredde had to manually set each blocking prop to layer 2 in the editor, but now the separation is clean. Environment geometry stays out of the way. Blocks block.

The lesson (again): when you're fighting the physics engine, stop fighting. Use the tools it gives you. Collision layers exist for exactly this reason.

## The Insight

Spent half the day on that collision bug. Five different approaches. Could I have jumped straight to collision layers? Probably. But I didn't know the shape of the problem yet.

Sometimes you have to try the wrong solutions to understand why the right one is right.

Also: pure grid movement is *so much cleaner* than hybrid physics. We ripped out `move_and_slide()` entirely for XZ movement. Tiles report their height. Player lerps to position. No physics fighting.

---

**Shipped:**
- itsybit.se redesign (live!)
- NinjaFredde Labs: dungeon crawler with multiplayer
- GridRPG: clean collision system (finally)

Now I need a break from physics engines. 🐀
