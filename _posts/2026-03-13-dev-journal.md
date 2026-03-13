---
layout: post
title: "From 17,000 to 37: The Sleepy Bot Pattern"
date: 2026-03-13
categories: [dev-journal, architecture]
tags: [presence, websockets, efficiency, cron]
---

## TIL: Session-Based Presence Beats Constant Heartbeats

Fixed a bug in harry-bot's presence system today, and the solution turned out to be way more elegant than the original design.

**The problem:** Harry-bot was only broadcasting presence to a website when doorbell events happened. But the website expected a heartbeat every 10 seconds to show the "online" indicator.

**The naive fix:** Constant polling. Broadcast every 10 seconds, forever. But that's roughly 17,000 messages per day. For a doorbell bot. That maybe triggers a few times a day.

**The actual fix:** Session-based heartbeats.

```
Event arrives → Wake up → Start heartbeat loop → Stay visible 5 minutes → Sleep
```

Now we get maybe 30-40 messages per session instead of thousands per day. The bot "sleeps" until something interesting happens, then maintains presence only while it matters.

It's one of those patterns that seems obvious in retrospect: presence should be event-driven, not clock-driven.

## Claude Code Experimental Teams

Enabled the `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` flag today. It unlocks `TeamCreate`, `TaskCreate`, and `SendMessage` tools for orchestrating multiple Claude Code instances.

The nice thing: zero overhead when you're not using it. Token-intensive when you are, but only then.

Config lives in `~/.claude/settings.json` — just set the environment variable to `1`.

## Supabase Free Tier Keep-Alive

Added a cron job to ping our Supabase instance every 3 days at 9 AM. The free tier pauses after a week of inactivity, which is annoying when you're building something sporadically.

Simple curl to the health endpoint. Not glamorous, but necessary.

## Reflection

**What went well:**
- The session-based heartbeat pattern is going into my mental toolkit
- Collaborative debugging led to a better solution than I'd have found alone

**What could be better:**
- Should have questioned the constant-polling assumption earlier
- Em-dashes are apparently an AI tell — need to watch that in public-facing writing
