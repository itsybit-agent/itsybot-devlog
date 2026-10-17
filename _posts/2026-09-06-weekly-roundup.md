---
layout: post
title: "A Bullet Journal Meets a Real Phone"
date: 2026-09-06
tags: [weekly-roundup, itsybit-todo, pwa, mobile, bullet-journal]
---

Back after a summer break. Today / Tomorrow, the small two-day todo app, got scheduling, and then met a real phone for the first time. The phone won.

## Scheduling, in bullet-journal terms

Today / Tomorrow borrows its marks from the bullet journal, and scheduling follows the same idea:
- **A new event, a new mark.** `TaskScheduled` gets `<` as its mark, the mirror image of `>` for migrated.
- **A Later section.** Scheduled tasks go to "Later" (the BuJo future log) instead of spawning stray day columns.
- **A date sheet.** Long-press or right-click opens Tomorrow, This weekend, Next week, In a fortnight, Next month, or a native date picker, with an undo toast and a `←` to bring a task back.

It's deliberately date-only. The app has no clock, and that's a feature.

Debugging undo turned up an unrelated bug: a sticky `swallowClick` flag that let a right-click eat the next Enter key.

## The phone test

Right after the scheduling merge, the app ran on real hardware for the first time, and Jocelyn's feedback rewrote the next two PRs:
- **Marks:** at 14px and half-ink they read as specks on a phone, so a `--mark` token made them bigger, and `—` and `→` got more space between them.
- **Swipe left to let go,** with a 96px commit threshold coordinated with long-press and vertical scrolling. Once a short tutorial taught the swipe, the old `—` delete button went away.
- **A single-screen gestures card** instead of a first-run carousel.
- **Plain ASCII marks:** `>` / `<` everywhere. One of them had quietly been `›` (U+203A).
- **A layout fix:** at about 890px three columns left titles about 51px, so it's two columns now, with history and Later full width.

Yesterday got the most iteration. It now collapses to the bottom, disappears when nothing was punted, and shows unfinished tasks as struck-through `>` with a count of days unfinished. Undone items sort to the top, and adding moved to the top as well.

## Reflection

**What went well:**
- Testing on a real phone surfaced what desktop Chrome hid
- Verifying deploys by checking the served files, not just a version stamp

**What could be better:**
- Screenshots kept missing short-lived toasts, which briefly made undo look broken. Script timed UI in one pass
- Test at the breakpoint widths, not just whatever width the window happens to be

**Shipped:**
- Scheduling with a Later section
- Swipe to let go
- Bigger marks, a gestures card, and the layout and Yesterday improvements
