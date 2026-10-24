---
layout: post
title: "Reading the Model Back Out of the Code"
date: 2026-10-24
categories: [research]
tags: [event-modeling, eventpad, barrio, verification, testing, agents]
excerpt: "How do you check what an AI built? We extracted barrio's event model from its code and compared it with the plan. Then we found the version that costs nothing."
---

Jocelyn put the problem plainly: "One of the things people struggle with is verifying AI output. But being able to extract a model from code cheap is quite good. Good way to spot diff from the plan, but also have it digestible by people."

The usual way to check what an agent built is to read the diff. That works for one change. It doesn't work for a whole app built over weeks by several agents in parallel, which is how barrio was made. Nobody reads 390 slices of C#. But everyone involved can read an event model: the timeline of things that happen, the commands that cause them and the views that show them. That's the form the plan was in to begin with.

So this week the EventPad session turned it around. Instead of checking the code against the plan, it rebuilt the model **from the code** and laid it next to the plan.

## What came out of barrio

Everything came from the source, not from the docs or the modeling board:

- **260 events**, read from the domain's event catalog, with fields taken from the record definitions.
- **393 slices:** 224 commands, 130 read models and 39 automations, across 81 chapters. They're ordered by when each event first appeared in git history, so the timeline follows the order things were actually built. Each slice notes the code file it came from.
- **1,440 Given/When/Then scenarios** read from barrio's rules tests. There were 3,804 in total; the 2,232 storage-level tests were left out so the result fits on a phone. One rule mattered a lot here: where a test only checked that a command was *accepted*, the agents had to guess which event it produced. Those guesses were thrown away rather than kept, and 588 scenarios are marked as having no confirmed outcome. A model that invents its own evidence can't be used to check anything.

## The differences were the point

Put next to the plan and the docs, the extracted model showed things nobody had noticed:

- **Three events in the catalog that nothing ever records.** `WeeklyLetterStarted` is defined but never created. The only route that would produce `LetterMovedToKitchenTable` now returns 410 Gone. `NotesSeen` has no writer at all.
- **Two retired events still in the catalog:** `StatusSet` and `StatusCleared`.
- **Events the docs list under a view that the code never actually feeds into it.** The docs say the view reacts to them; the code disagrees.

None of these are crashes. They're the quiet drift between what was planned and what exists, which is exactly what reading diffs one at a time misses.

## What it cost, and the cheaper version

The barrio run used roughly 2–2.5 million tokens of agent work: about 0.9 million for the slices and 1.3 million for the scenarios. About half the pipeline is plain scripts rather than agents. It's packaged as `tools/model-from-code` in the EventPad repo, with a `/model-from-code` skill, and rerunning it from the saved agent outputs reproduces the same model exactly, at no cost.

The bigger lesson came from what was expensive. Agents were needed because barrio's tests are written in many different shapes, so something had to read each one and work out what it meant. When tests share **one** shape, a script can read them directly:

- **Given** these events, **When** this command, **Then** these events.
- **Given** these events, **Then** this view.

So EventPad got 57 tests in exactly that shape: plain Node, about a tenth of a second, run in CI. EventPad's own model is now built from them with no agents at all: 11 chapters, 42 slices, 57 scenarios. Building it also flagged 13 event types that no test touches, and a real bug: adding a sticky note to a deleted slice still creates it.

That's the version that scales. If the tests are written as Given/When/Then from the start, the model comes out of the test suite for free, every time CI runs, and drift shows up as a diff in the model rather than months later.

## Digestible by people

The other half of Jocelyn's point was that the result has to be readable by people who don't read C#. The extracted model opens in EventPad as a board, an outline, or a picture you can share (Share → Outline, with or without the scenarios). Someone who planned the feature can scroll it on a phone and say "that's not what we meant" without opening the repo.

That's what verification needs: a format the person who knows the intent can actually check.
