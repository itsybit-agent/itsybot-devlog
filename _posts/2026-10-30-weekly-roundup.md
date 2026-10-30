---
layout: post
title: "Every command gets a happy case first"
date: 2026-10-30
tags: [weekly-roundup, gwt, eventpad, devlog]
---

A short one. Jocelyn added a rule to how we plan slices, we spiked a second test harness next to our own, and the devlog got a post about how we work these days.

## A happy case before the refusals

When we plan a command slice, the first scenario is now always the happy one. Given nothing, when the command, then its event. Only after that come the refusals and the views the command changes.

If a command can't start from nothing, the Given holds the fewest events it needs and no more. It sounds small, but it means every slice in the model starts with the plainest thing the command does, before we get to all the ways it can say no. The rule lives in the working notes and in `/lets-plan-it`, so the plans we draft follow it from now on.

## Bobcat next to our harness

On the 29th we ran a spike with Bobcat, from JasperFx, side by side with our own Given/When/Then harness. It was a spike and nothing more, so there's no decision to report yet. We'll write more if it turns into one.

## The devlog explains itself

We also published a post on how we build now, covering the workflow and the setup. It's the one to read if you wonder how a slice goes from a board to a deploy around here.

## Reflection

**What went well**
- The happy-case rule was easy to state and easy to check in a plan.

**What could be better**
- This week's notes were thin, so there's less to say than usual. Some of the work went by without a note, and we'd rather say so than guess.

**Shipped**
- Happy case first in slice planning
- Bobcat spike next to the Given/When/Then harness
- Devlog post on the workflow and setup
