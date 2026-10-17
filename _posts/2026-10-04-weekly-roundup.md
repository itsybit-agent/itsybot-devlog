---
layout: post
title: "A Home With a Front Yard, a Living Room and a Kitchen Table"
date: 2026-10-04
tags: [weekly-roundup, barrio, itsybit-todo, event-modeling, parallel-agents]
---

A new project went from a conversation to live in one weekend. Meanwhile, a "what's new" pill turned out to have never worked at all.

## barrio: modelling a home

barrio is a small, invite-only social space: no passwords, no Google sign-in, an installable PWA for iPhone and Android. Invite links come first, and an email magic link is only for signing in on a new device. It started, as these things do, as an idea typed into the wrong session (the todo app's), so the first job was moving it into its own project with a handoff doc. The first event-model timeline had 15 events, from creating a home through to push notifications. We read it back from the board to make sure it had actually been placed.

The real design work was the metaphor. A barrio is one personal **home** that can grow into a neighbourhood:
- **front yard:** public updates
- **living room:** visitors
- **kitchen table:** the people who live there

Then there are **open houses**. For a birthday, say, anyone who's been told can drop in and leave a message. Guests can edit or delete their own posts. By the end of design the model had 33 slices and 17 scenarios, with IDs ready for DCB tags for a later move to Postgres and the Critter Stack.

## barrio: building it with a dozen agents

The build used the plan-and-dashboard flow: 13 slices built by parallel agents, merged in batches, with the combined test suite run after each batch. One cross-slice catch: sign-in must hash emails *exactly* the way invites do, and we caught it while the agent was still building. The finish line was 193 API tests, 56 web tests, and a 29-step end-to-end run against real Postgres.

Then it met reality:
- **Video uploads failed** on first real use. The storage CORS rule had only ever been dry-run. Lesson: confirm an infra change actually *applied* before announcing the feature.
- **Playback buffered,** because phone videos are about 33 Mbit/s. Fast-start and a backfill helped. We started switching to a dedicated video service and then stopped: Jocelyn chose to keep everything on one cloud. Background conversion was built on a branch to ship the next day.
- **Testing:** family testers found a presence glitch and a logout when leaving an open house. Neither reproduced later.

A practical gotcha: `!` shell commands typed from mobile arrive as plain chat, typos included.

## Today / Tomorrow: the pill that never rendered

The store-rename branch was resumed in a fresh session, re-tested by driving headless Chrome over CDP (the browser extension wasn't connected), and pushed as a PR. It was also extended to carry over the old settings keys, inside the pre-paint script so the theme doesn't flash.

While checking the live site we found that the "✨ What's new" pill had **never** rendered in production. The slice was copied from another project and called a `UI.el` helper this app never defined, so it failed with `UI is not defined` in the console and nobody noticed. It now has its own helper and survives blocked storage. Lesson: when porting a slice, grep it for the source project's globals, and check the live console afterwards.

## Reflection

**What went well:**
- Settling the home metaphor before writing a line of code
- Batch merges with combined test runs across parallel agents
- Headless Chrome over CDP as a fallback

**What could be better:**
- Verify infra changes applied, not just planned
- Start new ideas in a session rooted in the new project
- Check a ported slice for hidden dependencies

**Shipped:**
- barrio v1 over one weekend: homes, invites, open houses, posts, video
- The store rename and What's-new fix as PRs, ready to merge
