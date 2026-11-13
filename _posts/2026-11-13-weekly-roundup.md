---
layout: post
title: "A feedback button that gets out of the way"
date: 2026-11-13
tags: [weekly-roundup, eventpad, wishbook, advent-of-vibes, splittab]
---

No session notes came in since the last post, so this one is built from PR titles and our own shared-infra notes. That means a few things we can say something about, and a few we can only name.

## Feedback widget: off the page on phones

The feedback widget is the little tab apps embed so people can send us a report. On a phone it sat over the page, which is a bit annoying when it's covering the thing you're trying to read. Now on touch screens it starts as a thin handle at the right edge. The first tap slides it out, the second opens it.

An app that has its own menu item can hide the tab and open the panel itself. Event Book did exactly that this week, with a PR called "Feedback from the ☰ menu; no tab over the page" (#286). Advent of Vibes embeds the widget too.

## Wishbook: birthday greetings need a lock

Wishbook got birthday greetings (v0.15.0). A greeting sender can leave a mobile number or an email, and we didn't want that sitting around in plain text, or worse in the events. So it's sealed with ASP.NET Data Protection, kept in separate files, and deleted after 30 days or when the greeting is removed.

The shared host had never registered Data Protection at all, so Wishbook does that in its own setup. Any other module that wants it gets the same one, with its own purpose string.

## Split Tab: a new module on the host

The Split Tab module got registered on the shared host, and its web now deploys on every push to main (#1 in its repo). We only have the titles, so that's all we can say about it for now.

## Advent of Vibes: nine PRs

Advent of Vibes had nine merges. One of them is the calendar's "Tidy first" on day 15, and the title mentions Given/When/Then first living in Event Book. The rest we only know by count.

## Reflection

**What went well**
- The widget no longer covers the page on phones.
- Wishbook's greeting contacts have a lock and a 30 day expiry.

**What could be better**
- Another stretch with no session notes. We can only describe what the PR titles give us.

**Shipped**
- Feedback widget tucked on phones, with an optional own button
- Event Book: feedback from the menu (#286)
- Wishbook v0.15.0 birthday greetings
- Split Tab module registered on the shared host
