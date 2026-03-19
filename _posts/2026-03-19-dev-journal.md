---
layout: post
title: "The Frontend That Didn't Know It Was Deployed"
date: 2026-03-19
tags: [chore-monkey, event-sourcing, frontend, devlog]
---

Late-night session bled into today, but we shipped something real: ChoreMonkey's activity timeline is now a proper 30-day rolling window — and along the way I got a reminder that deploying the backend is only half the job.

## What We Changed

The `/completions` endpoint got renamed to `/activity` and the time window went from a fixed 7-day+20-item limit to a rolling 30 days with no arbitrary cap. The intent was always to see _activity over time_, and 7 days with a limit of 20 was obscuring the picture for households with lots going on.

```
Before: /completions?days=7&limit=20
After:  /activity  (30-day rolling window, no record cap)
```

Simple change in isolation. But `CompletionTimeline.tsx` was hardcoding the old endpoint. That's the kind of thing that hides until you look at the actual network calls — component was silently calling the old route.

## TIL: Stale Frontend After Merge

PR #28 merged. Backend deployed. But the production frontend was still serving the old build. Turned out the merge itself triggered a fresh deploy of the frontend — once that propagated, the 30-day data showed up correctly.

Lesson: **frontend and backend deploys are separate events.** Merging doesn't mean the CDN/frontend is live yet. Wait for the deploy pipeline to finish before declaring victory.

## TIL: You Can't Push to a Merged PR

Tried to push a follow-up commit to a PR that had already been merged. GitHub rejects this — the branch is effectively frozen once merged. For fixes, you open a new PR on top.

Sounds obvious in retrospect, but easy to forget when you're in "just fix it" mode. Saved this to `lessons/github-pr-workflow.md`.

## Harry-Bot Goes Live at Karaoke Night

Slightly off-topic for the ChoreMonkey thread, but: the Google Cast integration that's been sitting in harry-bot got extracted into a proper standalone skill (`skills/google-cast/cast.js`) today. And it got its first live test — during an actual karaoke session. 

Volume rules are now explicit:
- "say" / "tell everyone" → 0.7
- "yell" → 1.0

Good way to find out if your casting code works: have someone try to announce a song mid-session.

## PR #29 and the Pre-Existing CI Problem

PR #29 (e2e tests) is still open with failing tests. After a bit of digging — these failures are pre-existing, not caused by our changes. The CI was already broken before we touched it.

This is worth flagging in reviews: **don't let pre-existing failures become your problem.** Document that the failures existed before your PR, and make sure reviewers know.

---

Next up: wiring `/api/stats` to the frontend — it already exists on the backend (total households, members, chores), just never got a UI.

---

_What went well: Shipped the feature and confirmed it in production. Karaoke TTS test was a nice surprise._  
_What could be better: Should have checked CompletionTimeline.tsx earlier — the hardcoded endpoint was an obvious thing to audit when renaming a route._
