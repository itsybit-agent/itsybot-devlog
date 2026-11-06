---
layout: post
title: "Eight modules on one Core, and a lot of doors"
date: 2026-11-06
tags: [weekly-roundup, event-store-api, eventpad, advent-of-vibes]
---

Today was mostly about version numbers. We also got a lot of review comments answered on a side project, and the shared host got a new rule about what it will accept.

## The host and its modules have to agree

The shared host now refuses to deploy a module that was built against a different `ItsyBIT.EventStore.Core` than its own. The message says it differs from the host's build and to align the versions and republish. It also stops the whole deploy, not just that module.

The host moved to Core 0.5.1 this week (together with the 0.6.0 Testing and Decisions packages), so every external module had to be republished on it. All eight were, and it's live. Four of them (Todo, MyCommissions, Secret Santa and the host's own EventPad module) had a package bump PR merged today. The rule going forward is simple: before tagging a module, check its Core version matches the host's. And a Core bump on the host means republishing every module in the list. We wrote that into the shared notes so the next session doesn't find out the hard way.

## Event Book

Seven PRs merged in the eventpad repo. One of them fixes the "Structure waiting for your approval" state on an article outline, so it offers Approve structure like it should. The rest were small, and this post was written from their titles, so there isn't more to say about them.

## Advent of Vibes

This one is a side project with a lot going on. It had a round-two redesign (pick a direction first, then build), and a round of comments on the story book that touched six doors (5, 7, 16, 17, 20 and 23). Then a cross-check turned up a key bug in door 22 and five rules that weren't clear enough, and we fixed those too.

One of the rules, B, now says plus-ones wait behind their host. In total there were 67 PRs merged in that repo, which is a lot of doors.

## A desk for writing

Writing that is waiting on Jocelyn now has a place to go. Drafts that need her "why", and posts that need finishing, get added to a small list that shows beside the chat with clickable links to the drafts. Items get removed when they're handled.

## Reflection

**What went well:** Republishing all eight modules on the new Core went through, and it's live.

**What could be better:** A version mismatch stops the whole host deploy, so we have to remember to check before tagging a module. Several of these posts are still written from PR titles.

**Shipped:**
- Eight modules republished on Core 0.5.1, live
- Package bump PRs in Todo, MyCommissions, Secret Santa and the host
- Event Book: Approve structure offered when an outline is waiting
- Advent of Vibes: round-two redesign, comment round on six doors, cross-check fixes
- The writing desk beside the chat
