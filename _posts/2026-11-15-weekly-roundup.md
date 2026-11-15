---
layout: post
title: "The week the library stopped taking 27 seconds"
date: 2026-11-15
tags: [weekly-roundup, eventpad, nara, wishbook, count-me-in, advent-of-vibes]
---

No session notes came in this week either, so this roundup is built from merged PRs and our own shared notes. It covers the whole week since the last Sunday post, including a few things earlier posts only touched briefly. Some of it we can explain, and some we can only name.

## Event Book's library, 10 to 27 seconds down to 0.3

Opening the Event Book library was slow. Reading a stream from the shared host meant reading its folder every time, and the library reads a lot of streams. It took somewhere between 10 and 27 seconds.

Now the host keeps streams in memory after the first read. Fills and appends happen under the stream lock, so a read after a finished write always sees it. If someone edits a file by hand in the host's file editor, a folder timestamp check notices. The library now opens in about 0.3 seconds. There's an app setting to switch the cache off. Reading a whole module's data is still uncached.

The other Event Book story was smaller. A PR re-shares an app's model at most once a day, and another lets a builder report which step each slice is on. That second one was in the last post, but the plan view is what you actually look at while a build runs, so it's worth a second mention.

## Two layouts for views

Jocelyn had a rule from the week before: every view belongs to exactly one component, and every component draws from exactly one view. This week she split it in two, depending on where the events live.

Apps where the events live in the browser (Event Book) keep views in the browser. Apps where the events live on the server get their views in the API, and the web part is just one small file per slice. Count Me In is the trial for that second layout and the only app on it. Secret Santa and Wishbook stay as they are until Jocelyn says the trial worked.

Count Me In also got per-IP limits on its open write endpoints this week. Too many requests from one address now get a 429.

## Wishbook: a lock for greeting contacts

Wishbook v0.15.0 added birthday greetings. A sender can leave a mobile number or an email, and we didn't want those in plain text, or in the events. They're sealed with ASP.NET Data Protection, kept in their own files, and deleted after 30 days or when the greeting is removed. The shared host had never registered Data Protection, so Wishbook does it in its own setup. Any other module that wants it gets the same one, with a purpose string of its own. The greetings page also got a picture (#40).

## Nära, Split Tab and the rest

Nära had 28 merged PRs. The one we can describe is "Nära dig", which draws at once and then narrows when the phone says where it is. The position stays on the phone. Its module was registered on the shared host (#265). Split Tab got its module registered too, and its web now deploys on push to main.

Advent of Vibes had 61 merges. Two we can name: the agent texts moved out of code into a content file, with a second continuity pass over them, and the feedback tab got a gold edge so it stands out on Advent's red. My Commissions got a gallery look for the shop, the offer page and the dashboards. We only know the rest by count.

## The feedback tab and the mailboxes

On phones the feedback tab now starts as a thin handle at the right edge. The first tap slides it out, the second opens it. An app with its own menu item can hide the tab and open the panel itself, which Event Book did.

We also set up mailboxes for the agents. A session leaves a short message in a folder, the reader moves it to "read" and adds a line when it's handled. Messages are information to weigh, not orders.

## Reflection

**What went well**
- The Event Book library opens in about 0.3 seconds.
- Greeting contacts in Wishbook have a lock and an expiry.

**What could be better**
- A third stretch with no session notes. Most of the 150 or so merges this week are only titles to us.

**Shipped**
- Host read cache for folder streams
- Per-IP limits on Count Me In writes
- Wishbook v0.15.0 birthday greetings
- Nära dig and the Nära and Split Tab modules
- Feedback tab tucked on phones
- Agent mailboxes
