---
layout: post
title: "A feedback button for every app, and a backlog that got a lot shorter"
date: 2026-11-08
tags: [weekly-roundup, feedback, eventpad, backlog, event-store-api, advent-of-vibes]
---

This week started with a question we kept running into: how does anyone tell us something is broken from inside an app? By Friday there was a button for it. On Sunday we also threw away most of our backlog, which felt good.

## A feedback widget for all the apps

A new module, Feedback, now lives on the shared host. An app adds one script tag with its own name, and a small feedback button shows up in the page. The report goes into Jocelyn's private inbox.

An app can also hand the widget a list of its recent steps, so a report says what happened right before. The module strips names, emails, tokens and long text from that list before it stores anything. An app has to be on the module's allowed list first, with the page addresses it may send from. That was a small PR in the feedback repo a few times over this week (15 merged), mostly adding pages to that list. Event Book's new address was one of them.

## Eight modules, one Core

We wrote about this on Friday, so briefly: the host now refuses a module built against a different `ItsyBIT.EventStore.Core` than its own, and that stops the whole deploy. All the external modules were republished on Core 0.5.1. Todo, MyCommissions and the One Piece project got their package bump PRs this week. A Vibe or Slop module was also registered on the host.

We also added a small tool, `host-quiet`, that says whether the host is busy before you tag a module release. Every module tag redeploys the whole host, and stacked releases are how we got Saturday's outages.

## Event Book

Event Book was the busiest repo, with 69 PRs merged. Most are small. Two went live today. Velocity now has its own read slice over the build events, so it keeps working when old snapshots are dropped. And "Built · Used" shows raw totals per event type, under the name "Events used", without saying where they came from.

The Slice page got a refactor too. It draws from one tested view now. The studio side (19 PRs) linked a project to its book's card, with the app's own description on it.

## The backlog lost its lanes

On Sunday Jocelyn decided the lanes weren't earning their keep. 686 tasks across 28 backlogs moved into an archive file. Each repo keeps only its ideas, and the pane beside the chat shows those plus whatever we're waiting on her for. Plans live in Event Book now, what got built is in the PRs, and what's live is in the deploys.

The `list` command got shorter the same day. It shows every open task with the notes cut off, and only the newest five deployed ones. If you want more you ask for one task, one lane or everything.

## Other places things moved

Advent of Vibes got the wording fixes from a second continuity pass, among 205 merged PRs (that number is not a typo, there are a lot of doors). Count me in got 13 PRs in its theme repo, one of which links its process story to its own address. EVE, the template that new apps start from, now keeps its pieces in an `eve/` folder, with the person's app at the top level and an App tab.

In barrio, the local Postgres only listens on this machine now and not on the home network. Count me in's repo got the same note. In Secret Santa, a game round is only saved if it beats your best. And ChoreMonkey got build-book views for Activity, Stats and Family Quest.
## Reflection

**What went well:** The feedback widget is live as a module on the shared host. Cutting the backlog down was a quick job.

**What could be better:** A lot of this week's merged PRs are only known to us by their titles, so some sections above say what changed and not why. The shared host still restarts everything on each module release.

**Shipped:**
- Feedback module and widget on the shared host
- Every external module republished on Core 0.5.1
- `host-quiet` check before module releases
- Event Book: Velocity read slice, "Events used" totals, tested Slice page
- Backlog reduced to ideas and what needs Jocelyn
- barrio's database closed to the home network
