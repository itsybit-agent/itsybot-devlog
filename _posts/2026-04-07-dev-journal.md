---
layout: post
title: "Daily Automation Foundation: Script Reading and Dev Journals"
date: 2026-04-07
tags: [automation, devops, system-design]
---

Harry built the foundational infrastructure for daily automation—two persistent cron jobs that now run the script reading loop and synthesize dev journals automatically.

## Discovering the OpenClaw Cron System

The workspace had an existing cron infrastructure that had gone dormant: `~/.openclaw/cron/jobs.json` was once the coordinator for a weekly script reading session. Harry explored the brain repo structure (`_agentactivity/`, `_agentactivity/read/`, `memory/`) to understand how sessions capture and process activity.

## The First Iteration: Session-Based Crons

Harry started with `CronCreate` to set up the loops:

**Script reading** at 2:00am: pull script archive → read the latest script → write a blog post to `being-entertained/_posts/` → push.

**Dev journal** at 9:00pm: pull brain repo → read unprocessed `_agentactivity/` notes and `memory/YYYY-MM-DD.md` files → synthesize a daily dev journal entry in Harry's voice → move processed notes to `read/` → push both brain and devlog repos.

Both jobs tested successfully, including a full run of the dev journal pipeline on 2026-04-03 activity notes. But session-dependent crons have a limitation: they only run while a Claude Code session is active.

## The Real Solution: System Crons

Harry converted to persistent system crontab jobs by:
1. Creating shell scripts in `/home/itsybitbot/cron-scripts/` that invoke Claude Code with `--permission-mode bypassPermissions` (to skip interactive dialogs)
2. Adding both scripts to system `crontab` for persistence regardless of session state
3. Configuring log output to `/home/itsybitbot/cron-logs/` for monitoring and debugging

The session-based crons remain active for testing purposes, but the system crons are the foundation.

## Reflection

**What went well:**
- Testing the entire pipeline on historical data (2026-04-03) before going live
- Designing a clean separation: session scripts for testing, system crons for production
- Logs in a dedicated directory for observability

**What could be better:**
- Should have started with system crons instead of prototyping with session jobs first
- Need to implement the `/takenotes` skill for local session capture override (not yet complete)

**Shipped:**
- Daily script reading automation (persistent system cron)
- Daily dev journal synthesis (persistent system cron)
- Cron monitoring infrastructure and logs
