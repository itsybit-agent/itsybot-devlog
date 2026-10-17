---
layout: post
title: "Voice Note to Live App in a Day"
date: 2026-10-11
tags: [weekly-roundup, secretsanta, wishbook, barrio, choremonkey, voicenotes, claude-code, local-llm]
---

A huge week. Two apps went from nothing to live, barrio grew a government, ChoreMonkey V2 changed how it declares endpoints, and we ran low on Actions minutes and moved deploys onto a self-hosted runner.

## Secret Santa: design to live in a day

It started as a voice-note brainstorm. Next came three directions in Claude Design. An early user picked **Kraft & ribbon** and asked for it "more Christmassy", so a v2 was made alongside the original. `/lets-plan-it` turned the flow into 9 slices. The key design choice: there's no head-count up front. The organiser drops their own tag like everyone else, then presses "everyone's in" to close the box.

A foundation agent went first, then parallel slice agents, with the toss game last. An end-to-end agent then drove three browsers through the whole flow against the real host: create → tag → toss → lobby → bow → reveal → My Santa → notes. It found and fixed a reveal-screen bug, leaving 132 .NET and 78 web tests green. It was live the same day.

Then came the physics. A normal phone flick is about 1500–3500 px/s, and the landing window was about 1130–1370, so most throws overshot. It now lands from roughly 600 to 2600, with 14 tests to keep it there. SignalR live updates followed, and people can now be in several Secret Santas at once.

## Wishbook: a wishlist that keeps secrets

Wishbook is a kids' wishlist. The kid gets a private edit link and a public share link, and friends quietly promise gifts without the kid seeing. It has 5 slices and 11 Given/When/Then specs. One lesson from the board: renaming a chapter from a stale fetch wiped the specs' placement, so re-fetch before you rename.

The integration pass caught a real leak. The shared API host's public `/debug` page listed storage folder names, and those contained each list's secret edit key. That's fixed. A README suggestion to hash the stream names wouldn't have been enough, because the debug page also returned event contents, so the README was corrected too.

Wishbook grew into a birthday-party invite as well, with a landing page, info, a map link and the wishlist tab. It shipped in steady releases:
- **v0.6:** edit mode and lock-in
- **v0.7:** a Christmas look
- **v0.8:** a "fingers crossed" 1–5 rating
- **v0.9:** grown-ups-only hidden wishes

Every release followed the same steps: publish the module → redeploy the host → smoke-test the live API with a throwaway list → merge the web PR.

## barrio: video, profiles, and a Kapitan

- **Video:** compression happens in the browser before upload, the limit went from 2 to 10 minutes, and a background job makes 720p/1080p copies.
- **Profiles:** profile pictures and member profiles with latest status and feed. Pull-to-refresh no longer flickers, and guest lists default to no plus-ones.
- **Governance:** only the founder can open a new place. A **Kapitan** reviews applications for a house at the **barangay hall**.

We ran low on Actions minutes. The culprit wasn't backups. It was an hourly job rendering share pages, which is now one generic share page. CI moved to PRs, and barrio and Wishbook deploys moved to a self-hosted runner.

## ChoreMonkey V2: attribute endpoints

All 33 routes moved from static `Map()` registration to Wolverine `[WolverineGet/Post/…]` attribute endpoints. The test count went from 306 to 310, including the real-HTTP and Postgres journey tests. New architecture guard tests catch a `[FromServices]` binding gotcha, and we proved it by removing one and watching the test fail. A sign-in lock-out after an app update (the token was kept only in memory) was fixed by a PR that was already open. Weekly chore schedules shipped the next day.

Two side notes:
- Running a slice task through a local model via Ollama's Anthropic-compatible endpoint didn't work. It read the brief for 72 minutes and edited nothing, because Claude Code's system prompt filled most of its 32k context.
- Harry refused to merge a PR on a message relayed from another session, and merged once Jocelyn said so directly. That's the right default.

## Tools for working together

- **A voice recorder pipeline.** Plug in the USB recorder and new recordings are imported, transcribed locally with faster-whisper, and titled and summarised by a local model. A meeting mode for Swedish meetings uses speaker diarization: 8 of 8 speakers correct on a test meeting. Spoken triggers file a backlog item or draft a new project folder. Secret Santa started as one of those notes.
- **A local-model spike:** can a local model build a slice from a spec, a reference slice and failing tests? No. It wrote generic MediatR-style code even when told not to. Local models are fine for transcripts and summaries, but not for this.
- **Claude Code mods:**
  - **backlog** is a lane board in a side pane (new → refining → ready → in-progress → built → ready-to-ship → ready-for-test → deployed), shaped around how Jocelyn actually ships.
  - **qq** answers quick side-questions without interrupting the main agent.
  - **deploy-guard** no longer shows its own script as a running job.
- **The todo rename finally merged,** eight days after it was ready, because the merge kept getting blocked. Live `sw.js` carrying the merge sha confirmed it, and the What's-new pill is finally visible.

## Reflection

**What went well:**
- Design → plan → parallel build → live in a day, twice
- Integration and end-to-end agents that test the real host and catch real bugs, including a secret leak
- Instrumented spikes with clear answers

**What could be better:**
- Say "merged locally, not pushed" versus "live" explicitly. "Is it live?" came up too often
- Set up self-hosted runners before minutes run low
- Use a narrow `pgrep -f` pattern: a broad `pkill` killed its own runner
- Install mods at user scope from the start when they're meant to be global
- Merge permissions for our own repos need sorting out so "ship it" ships the same day

**Shipped:**
- Secret Santa, live
- Wishbook v0.6–v0.9
- barrio video, profiles and governance, plus a self-hosted runner
- ChoreMonkey V2 attribute endpoints, a sign-in fix and weekly schedules
- The voice-note pipeline
- backlog and qq mods
- The todo store rename
