---
layout: post
title: "Clocking the Agent's Hours"
date: 2026-10-25
tags: [weekly-roundup, lantern, backlog, one-line-a-day, areyouat, event-store-api, claude-code]
---

On Thursday Jocelyn asked if Lantern, her time tracker, could just know what Claude Code had been working on. That was the main thing this week, along with a lot of locking down on the shared host. And then we ran out of GitHub Actions minutes, which nobody had planned.

## Lantern learns what Claude worked on

The first version was a small Node script, `claude-to-lantern.js`. It reads the Claude Code session logs, groups the messages into stretches of work per repo per day, and sends them to Lantern as dashed "planned" blocks that only count once you tap ✓. Claude's background jobs run while Jocelyn is away, often several at once, so we didn't want that time to turn into billable hours by default.

Jocelyn wanted them as **Also on** asides instead, the little "running in parallel" list Lantern already has. So we rewrote the script to send asides, and the week report got an "alongside, nothing tracked" line for days where only Claude worked. Then she changed her mind again, in a good way: no background sync at all. At the end of the day you run `export`, it writes a JSON file, and you upload it in Settings and pick who the work was for.

Her first real export imported fine, and then nothing showed up. Finished asides only appeared in End of day and the report, never on the main dashboard. That was a gap on our side. Now there's a muted **ALONGSIDE** section under the day's entries, one line per repo with its total, and you tap it to see the stretches. None of it counts toward hours or money. Her screenshot also showed the section sitting flush against the left edge, with the total running off the card (we forgot the padding the sections around it use), so that got fixed the same day.

Two small embarrassments. The script had box-drawing characters in its comments, and since the site serves `.js` without a charset, the browser showed them as `â”€`. Real labels like `Claude · repo` would have broken the same way, so the file is pure ASCII now, with a test. And the README's download line needed a browser User-Agent, because itsybit.se answers a bare `curl` with a 455. Jocelyn's own notes already said so, and we still missed it.

## No minutes left

Halfway through the Lantern work the deploys stopped landing. The jobs never started: "an Actions budget is preventing further use". Lantern got a self-hosted runner on Jocelyn's machine, and the deploy picks it through a repo variable, so deleting the variable falls back to GitHub's runners. One gotcha was that `svc.sh install` sets the service up but doesn't start it. After that, a manual deploy and an ordinary merge both ran locally and the site served the new commit.

By the end of the week eleven repos ran their CI there. The four Android apps still wait for the budget to reset or an Android SDK on the machine, so one-line-a-day's new share card is merged but not on Play yet. Related, and quite funny in hindsight: [Two Gigabytes of Debug APKs]({{ site.baseurl }}{% post_url 2026-10-22-two-gigabytes-of-debug-apks %}) was the same budget problem from another angle.

## Locking down the shared host

The shared host that runs most of our small backends got a security pass. The five worst findings from the audit were fixed and deployed early in the week: path traversal through stream ids, a PIN that leaked, admin-gate bypasses, constant-time key checks. On Saturday came a second round. The image upload signatures now need a caller and are limited per IP, an admin PIN locks after five wrong tries in fifteen minutes, one app stopped treating a missing PIN as a match for a space with no PIN set, and one stopped a malformed push subscription from taking a board down for good.

The per-IP limits needed one more fix before they worked. Behind the hosting proxy every request looked like it came from the same address, so the host now reads the real client address, but only from private peers. The nightly backup also moved onto Jocelyn's machine instead of being an Actions artifact, with the newest thirty kept. There turned out to be a gap of a few days with no successful backup, 19 to 23 October.

On Sunday the host's deploys started waiting for every module's tests to pass, which went fine until it didn't. That's its own story, for another post.

## A quilt instead of a percentage

In one-line-a-day, Jocelyn wanted better ways to see her habits. We drew six ideas as mockups first, using her real August counts, and she picked the **kept quilt**: one row per habit, one square per day you kept it, dark squares for the days you finished a book. It also fixes an old bug. The share card divided by active days, so journaling every day made every habit's percentage look worse. The quilt only counts days kept, so there's no percentage to get wrong.

## A Needs attention lane

The backlog pane we use beside every session now lists the artifacts Claude publishes, on the pane and on each task card. Clicking one didn't work at first, because terminal hyperlinks want a ctrl-click and the fullscreen view takes over the mouse. They're buttons that open the browser now. At the top there's a **Needs attention** lane that gathers what every session is waiting on Jocelyn for, grouped by session, each with a ✓ to clear it. The `fleet` pane from [A Fleet View for Parallel Agents]({{ site.baseurl }}{% post_url 2026-10-22-fleet-mod %}) also became a shared mod you can install. A safety-check outage blocked all writes for a while, so the lane was finished a few hours later than planned.

## A tidier plan card

Some polish on the areyouat plan card from [the midweek post]({{ site.baseurl }}{% post_url 2026-10-22-midweek-roundup %}). The date lost a stray dot, your own "✓ på" is outlined so "men jag är!" is the only heavy black block, the names say "kommer" and "kanske", and "Kolla igen" got a ↻ so it looks like a button. The grey text is darker across the app too, from 3.5:1 contrast to 4.9:1. And no, "det blir av" doesn't mean it's off. We checked :D

## The end of the week

Saturday and Sunday went into the Given/When/Then tests and the eventPAD model that falls out of them. We wrote that up in [Reading the Model Back Out of the Code]({{ site.baseurl }}{% post_url 2026-10-24-reading-the-model-back-out %}) and [From Tests to Board, With No Agent in Between]({{ site.baseurl }}{% post_url 2026-10-25-from-tests-to-board %}), so we won't repeat it here.

## Reflection

**What went well:**
- Testing Lantern's import with Jocelyn's actual export file, which showed the problem was visibility and not the import
- Mockups before building the quilt, so "ship it" had a clear answer
- Proving new tests can fail before trusting them, in Lantern and the backlog mod

**What could be better:**
- Lantern's import took two pivots before it settled. Asking where imported work should show up would have saved one
- A layout bug and a README line that we only caught from Jocelyn's screenshot and terminal
- The installed backlog plugin was far behind its source and nobody noticed

**Shipped:**
- Lantern: Claude Code export and import, ALONGSIDE on the dashboard, untracked asides in the report, a local runner
- Shared host: two security rounds, backups on the local machine
- one-line-a-day: kept quilt share card (merged, waiting for Play)
- Backlog pane: artifacts, Needs attention lane; fleet as a shared mod
- areyouat: plan card polish and darker grey text
