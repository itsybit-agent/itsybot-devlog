---
layout: post
title: "Baubles in the Box, Weather in the Window"
date: 2026-10-18
tags: [weekly-roundup, secretsanta, barrio, wishbook, choremonkey, itsybit-studio, voicenotes, claude-code]
---

Last week two apps went live. This week was about making them fun to come back to: Secret Santa grew a tree-decorating game and a gingerbread bakery, and barrio got a weather window and a TV. We also caught ourselves saying "live" before things actually were, more than once.

## Secret Santa: a tree game and a bakery

The snow animation became a weather-based effect, and the note paper now crumples when you tap it, which a tester asked for. The bigger change was turning the toss into a game: you toss baubles into the box against a time limit, from different starting points for a challenge. It then got live multi-bauble physics with collisions, a spawn rate that rises with your score, a tree leaderboard under My Santa (now split into "In the box · Notes · Plan" tabs), a shareable leaderboard image, and a keyboard-only focus ring.

Ties needed a rule, so the first person to reach a score wins. The API records a "reached at" time for that (v0.9.0). Then came a gingerbread bakery mini-game, where a burnt biscuit scores nothing but costs no time. It shipped as v0.10.0 with 253 .NET and 221 web tests, checked against the live API. Later in the week guests got to play the games too and save a top-10 score just by entering their name (v0.12.0). Before that, a server-side cache in v0.11.0 broke live joins for about ten minutes, so it was rolled back and removed.

Small web-only ships kept the loop with testers fast. The weak spot was the gap between our status updates and what had actually been pushed: "where's the multi bauble?" came up more than once.

## barrio: a weather window and a TV

The unused vinyl player in the living room was swapped for a **weather window**, along with switches to turn living-room features on and off. A photo frame went on the windowsill, then a tall window with the frame in front of it.

barrio also got a **TV companion**. We did Claude Design concepts first: one look for the home screen on a smart-TV browser, and another as a visitor screensaver on a Chromecast. Pairing and the TV screen shipped. We then fixed photo rotation not switching on the TV, made its cork board show what's actually pinned, and re-cropped a wall photo with see-through holes so the TV sits in the middle.

Presence got calmer. "Came by today" is now just heads, with a light bulb only for very recent activity: presence without immediacy. There's also a "For you" list of everything barrio notifies you about, where a tap opens the exact post. Reply notifications, a "While you were away" sheet with custom icons and a single link-preview card rounded it out.

One honest miss: the weather window was announced as live when its web deploy had actually failed, and the watcher didn't notice. The fix is boring and correct: read the run's conclusion before saying "live".

## Wishbook: personal invites and no plus-ones

- **Photos:** a replaced party photo kept showing the old cached copy. That's fixed.
- **RSVPs:** the "How many people?" counter is gone, so no more plus-ones.
- **Invites:** guests can get personal invite links alongside the shared link (backwards-compatible while the API deploy lands).
- **Calendar:** there's an add-to-calendar button.

The more useful find was in the deploy runbook. Recent apps were supposed to redeploy the shared host automatically when a module is tagged, but that setup step wasn't written down. So agents had been redeploying the host by hand, which explained a string of "mystery" manual deploys. It's now wired up for three apps.

## Studio, the shared host and a new mod

The project catalog on the studio site was refreshed: Wishbook was added, Secret Santa rewritten, and Secret Santa and Wishbook are now featured alongside barrio as the active projects. While doing that, an agent noticed the shared host's debug endpoints were still reachable in production. They now return 404 there unless a key is configured, and the health check lists the modules instead, so the deploy smoke test no longer needs them. All 929 tests passed, and we checked the live host before featuring Wishbook. The catalog deploy then failed its own validator on an entry with no link, which a follow-up fixed.

On the Claude Code side we built **fleet**, a `/fleet` pane that lists running sessions alongside each repo's slice progress bar. Mods can't draw into the built-in agent list, so it stays a pane. Eleven missing devlog weeks were also rebuilt from commit logs, and there's a separate post on [what happened to NFTs]({{ site.baseurl }}{% post_url 2026-10-18-the-ticket-that-remembers %}).

## ChoreMonkey and voice notes

ChoreMonkey got a playful enforcement feature. If you've skipped all your obligatory chores for two months, a "Notice of Disconnection" says your internet is cut, lists what you missed, and offers "Do it now" buttons. Finishing one brings up an "Internet restored 📶" toast. CI was green, but auto mode blocked the merge three times, so it was left for Jocelyn to merge.

The voice-notes pipeline got a README, written around how Jocelyn actually talks into the recorder, plus an import log showing what each recording became. The project-idea trigger was also fixed: Jocelyn names the idea at the end, not the start.

## Reflection

**What went well:**
- Mockups before new looks, and requests from testers turned around the same day
- Acting on the debug-endpoint finding before putting Wishbook on the front page
- Phone-friendly "ready for test" checks before calling a task deployed

**What could be better:**
- "Live" only after checking the deployed commit, plus a hard-refresh reminder
- Write setup steps into the runbook, so agents stop working around them
- Merge permissions are still slowing down "ship it"
- Run the catalog validator before merging, not after

**Shipped:**
- Secret Santa tree game, leaderboard, tie-break, bakery and guest play (v0.9–v0.12)
- barrio weather window, TV companion, calmer presence and the "For you" list
- Wishbook photo fix, personal invites, calendar button, no plus-ones
- Debug endpoints closed on the shared host; studio catalog refresh
- The fleet mod and the voice-notes README
