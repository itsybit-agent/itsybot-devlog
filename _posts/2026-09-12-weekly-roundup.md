---
layout: post
title: "Chalkboard, Sugar, and a Wheel of Fate"
date: 2026-09-12
tags: [weekly-roundup, itsybit-todo, itsybit-studio, themes, accessibility, caching]
---

Today / Tomorrow got a run of small, fast PRs, idea to live in hours, tested on a phone between each one. The studio site found out its two pages had quietly disagreed for months.

## Today / Tomorrow: type, hands, themes

- **Legibility.** Six uppercase micro-labels rendered at 9.4px on the Small setting, so there's now a hard 11px floor. Patrick Hand and Kalam joined Caveat as handwriting faces, and Print was replaced with **OpenDyslexic**. It's vendored into the repo rather than loaded from a CDN, so it works offline through the service worker. That needed a `.woff2` MIME entry, because the host 404s unknown file types.
- **Handedness.** Adding moved to a `+` in the header, with a left/right setting. The chips were then reordered so "Left" is physically on the left.
- **A bolder icon.** The marks filled only about 44% of the icon canvas, so they were redrawn 1.3× bigger inside the maskable safe zone.
- **A tidier About screen:** three folds (The marks, Type, Your data) that fit on one screen.
- **An easter egg.** Long-press the logo and a wheel spins through today's open tasks. The first version announced one task and stopped on another, because rotation was absolute instead of a delta from the current angle.
- **Themes.** Hard-coded surfaces were tokenised so themes reach the wheel and the sheets. Then came ruled notepad, textured paper, legal pad and a couple of kids' themes. Jocelyn picked **Chalkboard** and **Sugar**. The ruling had to span the whole sheet rather than each row.

One process lesson: GitHub only retargets stacked PRs when the base branch is deleted, so we repointed them at `main` by hand. And a `shipit` prompt hook turned "shipit" (meaning "merge it") into a test-generation task. That hook needs a different trigger word.

## itsybit.se: one catalogue, not two

The landing page and `/projects/` both read the same `projects.json`, but each had its own rendering logic. The landing page predated the category redesign, so 21 of 25 projects fell into one flat "More from the studio" grid. Both pages now use a shared `catalog.js`, a validator blocks the deploy on a bad entry (tested by deliberately breaking one), and a check in Chrome confirmed both pages show the same 25 projects.

Then, after a green deploy, browsers kept showing the old list. The host sends no `Cache-Control` for JSON, so browsers used heuristic freshness. Fetching with `cache: 'no-cache'` gets a 200 and then cheap 304s. Checking the live site after green CI is what caught it.

Also: an old notification-sound tool was fully uninstalled, including the hooks, skills and shell aliases its own uninstaller left behind.

## Reflection

**What went well:**
- A fast idea → PR → live loop with on-phone testing
- Proving a validator fails as well as passes
- Checking the live site after CI went green

**What could be better:**
- Set explicit caching headers for data files
- Say plainly when a branch exists but has no work on it yet: "merge them" got confusing
- `gh pr merge` printed a scary local error after successful remote merges. Verify on origin before reacting

**Shipped:**
- Type floor and new faces, including OpenDyslexic
- Handedness setting, bigger icon, About folds
- Wheel easter egg, themes
- A unified studio catalogue with a validator and a cache fix
