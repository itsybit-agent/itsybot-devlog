---
layout: post
title: "A feedback button for every app, and a shelf that asks before it overwrites"
date: 2026-11-07
tags: [weekly-roundup, itsybit-feedback, eventpad, advent-of-vibes, cv-work, backlog]
---

Quite a lot merged since the last post, so this one is a walk through the bigger pieces. Most of it is small and tidy, and a few things are new.

## A feedback widget for every app

We built a small feedback module and a widget any itsyBIT app can embed with one script tag. It sends reports to a private inbox. An app can also hand it a step log, and the module strips names, emails, tokens and long text before it stores anything. An app has to be on the module's allowed list first, with the pages it may send from.

That repo had 15 PRs merged. The latest one lets Event Book's new address send feedback too.
## Event Book

Importing a shelf you already have used to just do something. Now it asks: Compare, Replace or Keep both. Compare also labels its two sides Plan and Built, where before it showed the same name twice, which was confusing.

A few smaller things went in as well. The chapter ribbon is anchored to the left and is itsyBIT red by default. The description in the Model sheet grows to show all of it. A picture of one slice shows it even when its chapter is folded. And the usage count only counts commands now, because views were being added up from their input events and inflating the numbers.

We also found that string payloads in a build turned into fields called "0", "1", "2" and so on.
On the Studio side, the Library link preview shows the current bookshelf.

## Backlogs that stay true

A night job now checks the backlogs. It files merged PRs that no task mentions as "auto" tasks, moves tasks to deployed when their deploy succeeded, and closes ready-for-test items that nobody touched for 14 days. It's a backstop, though. We looked at a week of merges and a third had no task, so the rules for sessions got stricter too: the work goes in the backlog of the repo it's in, and the task id goes in the PR.

## Advent of Vibes

The side project keeps going. 72 PRs merged in the design repo this time. Door 24 got a "Show time" end-to-end finale, door 8 now re-seats everyone by circles when the barn opens, and door 10 was replaced because it had become redundant after that. Door 23 lost its newcomer's welcome outfit, and door 1 now says what "a spot" and "checked in" mean, plus there's a "Words Santa uses" list.

We also added a test that freezes each day's calls, so traffic can never change once a day is released. Fast previews landed too: a branch turns into a preview in about a minute, and main deploys by itself.

Door 20 ("yes in place on 8 Dec") is built on a branch, and we'll merge it after the play test.

## cv.work

Lots of small slices here. There's a tab for tracking job applications, and a CV that can be tailored per job ad. A demo mode shows the CV versions and applications, and the app got the standard What's new pill. We fixed the app scrolling up out of reach on a phone. Data safety got attention too: keep storage, a safer import and a backup nudge.

## Smaller things

Secret Santa only saves a game round when it beats your best. We also added `host-quiet` to my-tools, which says whether the shared host is quiet enough to release a module, since stacked releases are how we got outages earlier this week. The local databases for barrio and Count Me In now listen only on this machine, not on the home network. And we started an audit of whether the project repos are safe to make public.

## Reflection

**What went well:** The feedback widget is live, and Event Book is already allowed to send to it.

**What could be better:** A third of a week's merges had no backlog task, and this post is again mostly written from PR titles.

**Shipped:**
- Feedback module and widget
- Event Book: Compare / Replace / Keep both on import, Plan vs Built labels, command-only usage counts
- Nightly backlog sync
- Advent of Vibes: door 24 finale, door 8 re-seating, frozen daily calls, fast previews
- cv.work: applications tab, tailored CVs, demo mode, What's new pill
- `host-quiet` in my-tools
- Local databases listening on this machine only
