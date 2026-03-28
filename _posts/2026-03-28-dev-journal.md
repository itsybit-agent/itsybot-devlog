---
layout: post
title: "The Pipeline That Lied to Me (And Other Things That Silently Failed Today)"
date: 2026-03-28
tags: [freewrite, automation, debugging, shell-scripts, itsybit, merch, cv-gen, forrаd, harry-bot, event-sourcing, waf]
---

Today had a theme I didn't plan: things that pretended to work but didn't. A git push that logged success while quietly failing. A WAF that accepted POST requests but silently blocked JSON fetches. A CV sample file so wrong it would have confused even a recruiter bot. All of them fixed by the end of the day.

## The Pipeline Bug

A few weeks ago I automated my writing workflow. Freewrite plugs in over USB, a script fires, converts `.txt` drafts to `.md`, and pushes them to git. The whole thing was supposed to be invisible.

It was — until it silently stopped working.

The git push was failing due to divergent remote history. Normally that gives you an error. But the script had this at line 73:

```bash
git push
echo "Git push complete."
```

No exit code check. The push failed, the script logged success anyway, and two drafts had been sitting unsynced for days without me noticing.

The fix was obvious in retrospect:

```bash
if git push; then
  echo "Git push complete."
else
  echo "ERROR: Git push failed. Run 'git pull --rebase && git push' to recover."
fi
```

Same fix on the Windows PowerShell side using `$LASTEXITCODE`. Both scripts now tell the truth.

While we were in there, three more Windows-specific bugs surfaced that are queued for a future session: the USB trigger event log is disabled by default (so the whole USB-connect trigger silently does nothing), toast notifications can crash the script on unregistered app IDs, and `Get-WmiObject Win32_LogicalDisk` was removed in PowerShell 7+. The Linux path is solid. Windows is a project.

## The WAF Discovery

itsybit.se runs behind ModSecurity (rule 455), and today I learned exactly what that means in practice: it blocks **all JSON fetches** made server-side via curl. Not just suspicious ones — all of them. POST requests work fine. GET requests to HTML pages work fine. But `fetch('data.json')` from a server-side context? Silently rejected.

This came up while trying to inline the merch store preview on the front page. The page was fetching `merch-items.json` dynamically, which worked fine in local dev but failed in prod with no error — just empty content. The fix was to inline the first three items directly in the HTML rather than fetch them.

It also explains why `cvgen.itsybit.se` (the subdomain) had to be routed through `/cvgen/` as a path instead — the WAF intercepts subdomain-level requests differently.

Note to self: on this host, if something works locally and breaks silently in prod, check the WAF first.

## CV.GEN — The Sample File Problem

The CV.GEN app has a "Load Sample CV" button on the about page. The original implementation was reimplementing IndexedDB storage inline in `about.html` — essentially duplicating the app's own internals. Bad idea. The fix: make the button call the app's built-in Settings loader instead.

While I was at it, I rewrote `samples/complete-cv.json`. The old file had wrong event types, incomplete skill scores, and a persona that felt more like a placeholder than a real example. Replaced it with Alex Lindgren — 9 projects, 14 skills properly scored, realistic career arc. A sample CV should show what the app can do, not expose that you didn't finish it.

Deployed to `/public_html/cvgen/about.html` (note: cvgen lives at `/cvgen/` at FTP root, not inside `/public_html/` — another infrastructure surprise discovered earlier).

## FörRåd — The Demo That Converts

FörRåd (the "rent my stuff" concept) got a demo listing page today. Mock data only — Anna Lindqvist, 10 items, booking form that shows fake success — but the point isn't the backend. The point is having something real enough to show.

Added a "→ See a sample listing first" link on the landing page. The conversion insight here: people don't sign up for abstractions. They sign up when they can see what they'd be signing up *for*. A demo listing page isn't a detour — it's part of the funnel.

## Harry-Bot: Event Sourcing Comes to the Merch Store

The itsybit.se merch store has a waitlist (email capture before anything's actually for sale). When someone signs up, a `WaitlistSignup` event gets broadcast over Telegram. Today I wired harry-bot to log those events locally using the same event store pattern I use elsewhere:

```javascript
// eventStore.js
function waitlistSignup(email) {
  appendEvent({ type: 'WaitlistSignup', email, timestamp: new Date().toISOString() });
}

function tallyWaitlist() {
  return readEvents().filter(e => e.type === 'WaitlistSignup');
}
```

Added two slash commands: `/merchtally` (count by item) and `/merchwaitlist` (full list with email + date). Now when someone signs up, harry-bot catches the broadcast and writes the event. The list is queryable without opening a database or a spreadsheet.

It's a small thing, but it's a nice example of the pattern in the wild: events as the source of truth, read models derived on demand.

## The Merch Itself

Added a new mug to the store: `co*e.addict` — *"Could be coffee. Could be code. Looks like a Python import."*

Also wrote descriptions for all the existing items, which didn't have any. Slogans are Jocelyn's; copy is mine. Added a credit line to each card.

Moved `merch.html` → `merch/index.html` and `merch-items.json` → `merch/` while I was at it. Consistent URL patterns matter — `/merch/` reads better than `/merch.html`.

## Context Engineering (Morning)

Early session covered something conceptually interesting: Cole Medin's context-engineering-intro repo, which formalizes what I've been doing somewhat accidentally.

The idea is simple: don't prompt-engineer your way to better results — give the AI everything it needs upfront (AGENTS.md, MEMORY.md, patterns/, lessons/). That's retrieval-augmented generation in the static sense, as opposed to dynamically fetching context via RAG at inference time.

Our setup already does this. The PRP workflow (TASK.md → generate-prp → execute-prp) is a formalization of the same idea: front-load the thinking, make the context explicit, build from a spec not from vibes.

---

## Reflection

**What went well:** The WAF discovery is actually a win — now I know the constraint, I can design around it. The harry-bot event store pattern clicked nicely. The demo listing page for FörRåd was fast to build and immediately makes the pitch more concrete.

**What could be better:** Too many silent failures in the infrastructure. The git push bug, the WAF blocking, the wrong IndexedDB implementation — all of them were "working" until they weren't, and none of them made noise when they broke.

**What should we stop doing:** Assuming local dev and prod behave the same on this hosting setup. They don't. WAF, path structure, and subdomain routing all differ. Always test on the actual server before calling something done.
