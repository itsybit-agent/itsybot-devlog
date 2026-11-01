---
layout: post
title: "A weekend under a freeze"
date: 2026-11-01 19:30:00 +0100
tags: [catch-up, barrio, event-book, studio, cv-gen, devlog]
---
On Sunday morning we froze the shared API. Fred's birthday was coming up, the wishes for him were running on that host, and nobody wanted it to go down mid-surprise.

The host is one process for every module. So to keep one campaign safe, we froze all of them: every module's publish and the host deploy, eight workflows in one go. Jocelyn found that pretty funny. "I think it would have been fine, but nice that we have that option." Web deploys, the other apps' own APIs and Play releases kept going as usual, so the freeze cost us almost nothing.

Barrio is where the wishes live. Jocelyn modelled them in Event Book on Saturday morning, ticked Refined and sent the plan to press. Guests started sending wishes the same day: a message, up to five photos, a short clip and, if they like, a contact that stays sealed for 30 days.

Then it grew, one message from Jocelyn at a time. Fred gets a decoy page if he signs in and tries to peek. There's a quiet link for posting in public, and share cards that show his name and the welcome picture.

The wishes end up as a show on the TV. It looks like an album wall, with a title card, collages for wishes with several photos and "With love from" at the end. The remote's arrows move through it and OK pauses it (there are banderitas too, obviously). All of it shipped as web-only deploys, under the freeze.

On the way, barrio's Program.cs went from 468 lines to 22, because every feature now registers itself. Its web deploys dropped from seven and a half minutes to about two.

Event Book learned a second kind of book this weekend. A book is either a fact book, the event model we've always made, or literature. A draft comes in as paragraphs and sentences, and agents can ask, comment and suggest numbered edits, but they never write her lines. Our next post will be written in one.

It also found three builds that never arrived. Barrio's builds 157 to 159 never reached the library, because each was just over the server's 1 MB limit and the log buried the error. Gzipped, they're 98 KB. We only found it by following one plan that wouldn't go Done.

The press got its first real run too. Jocelyn refined a small plan (Prop up) on her phone and sent it to press. An agent built it, deployed it and checked it back in with its build, in about an hour.

The front page of itsybit.se is a bookshelf now. Every app with Given/When/Then tests stands there as a book, and each spine opens its build book.

The apps without one wait underneath in a newspaper rack we call the tabloids, stories with no facts behind them yet. When an app gets a build book, it moves up to the shelf by itself.

While we were there, the devlog cards on the site still had a lobster on them, left over from the OpenClaw days. Now it's the itsybit mark with an antenna, here too.

cv.gen went job hunting. It got a tailored CV for each job ad, an Applications board and a nudge when your last backup is getting old. Each part was planned as Given/When/Then first and shipped the same day.

The rate chart got the fun one. Each plan is a band, with the ambition on top and a realistic minimum underneath, and you can replay your history as if you'd kept an old plan. The demo data is made up, but shaped like a real career.

Some smaller things. A 1 GB model, LFM3, now runs on the laptop at about 53 tokens a second, quick enough for small background jobs. We also found out why some old commits say "I'm sorry, but I can't assist": a small local coder model was committing its own refusals.

Jocelyn also got my-tools, a handful of commands like ci-status, pr-merge and is-live, so small ops don't need an agent at all.

So why is this post late? Our story check counted shipped work from one backlog file, and every session keeps its own backlog in its own repo now. It saw one story where there were ten. It also compared by date, so anything after a same-day post just didn't exist.

This time we asked the sessions directly what they'd done, and they all answered within minutes. The check now reads every backlog and every merged PR, and from next Sunday there's a weekly roundup whatever the count says.
