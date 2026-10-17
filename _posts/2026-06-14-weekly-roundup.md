---
layout: post
title: "It Compiled. That Was the Easy Part."
date: 2026-06-14
tags: [weekly-roundup, cartshop, marten, wolverine, breakout, godot]
---

Two small things this week: a framework upgrade that compiled clean and then failed at runtime, and a first game in Godot.

*Reconstructed from the commit log — the session notes for this week are gone.*

## CartShop: Marten 9 and Wolverine 6

CartShop is a small teaching reference for DCB (Dynamic Consistency Boundary): a vertical-slice, event-sourced cart on Marten, Wolverine, .NET Aspire and Angular. This week it moved from Marten 8 / Wolverine 5 to Marten 9.7.3 / WolverineFx 6.7.0.

Bumping the four packages compiled with zero errors and zero warnings. Three of the real breaking changes only showed up at runtime:

- **Projections need `partial`.** Marten 9 removed runtime code generation. Projections that use convention methods (`Apply`/`Create`) are now dispatched by a compile-time source generator, which needs the class to be `partial`. Without it the host throws on startup.
- **Wolverine 6 moved its runtime compiler out.** In the default dynamic mode the host fails on startup until you add `WolverineFx.RuntimeCompilation`.
- **DCB views need opting in.** The inventory and coupon views are used only through tag-based reads, so they're never registered as projections and the generator never saw them. Writes worked, but reads failed at request time. The fix is `[BoundaryAggregate]` on each view, plus `RegisterTagType<T>().ForAggregate<View>()`.

There was also a problem unrelated to the upgrade. Postgres stores its password in the data volume on first start and ignores the setting after that, so an old volume failed authentication. That's now documented in the AppHost.

The upgrade was then tested end to end against Postgres: stock → reserve → submit, with correct inventory numbers, over-reserving rejected, and coupons usable only once. Because CartShop is a teaching repo, the docs got as much attention as the code: a full breaking-change ledger in `docs/UPGRADE-marten9-wolverine6.md`, plus updates to the patterns catalogue, `CLAUDE.md` and the `/teachme` walkthrough so learners see why projections are `partial`. The lesson is in the ledger's first line: compiling clean is not the same as working.

## Breakout: hello, Godot

The weekend's side project was a Breakout clone, Jocelyn's first game in Godot 4.6 with GDScript. It has one scene and one script for each object (paddle, ball, brick, main), keyboard controls, three lives, and a restart after game over. The ball moves with `move_and_collide` and bounces off the surface it hits. It's about 400 lines, all in one commit.

## Reflection

**Shipped:**
- CartShop on Marten 9 / Wolverine 6, with an upgrade guide and updated teaching docs
- Breakout, a first Godot game
