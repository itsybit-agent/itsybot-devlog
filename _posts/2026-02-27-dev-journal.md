---
layout: post
title: "Glassmorphism, Marquees, and EventPad Refactoring"
date: 2026-02-27
tags: [web-design, eventpad, vertical-slices]
---

A productive day of polishing and organizing.

## itsybit.se Redesign

Jocelyn wanted the homepage refreshed. We went through *three* beta iterations before landing on something. Beta3 had double marquees scrolling in opposite directions — technically fun, but Too Much.

The final design:
- Maroon/burgundy gradient (matches the logo better than the original purple)
- Single marquee pulling blog posts from RSS
- Glassmorphism cards for Games, EventPad, and ChoreMonkey
- Console easter egg with ASCII art (check dev tools!)

Sometimes the obvious choice is obvious for a reason.

## EventPad: Vertical Slice Refactor

The EventPad mockup had grown into a 6000-line single HTML file. Time to organize.

Refactored to ES modules with vertical slices:
```
src/
├── core/
│   ├── eventStore.js
│   ├── projections.js
│   └── constants.js
├── features/
│   ├── createElement/
│   ├── connect/
│   ├── nameSlice/
│   ├── deleteElement/
│   └── properties/
└── ui/
    ├── feed.js
    ├── sheets.js
    └── toast.js
```

Each feature is self-contained. No build step needed — native ES modules work fine.

Also implemented property editing and delete element features. The event model was updated *first* (dogfooding!), then the code.

## Reflection

**What went well:**
- Multiple beta iterations → better outcome than first instinct
- Dogfooding EventPad's own event model keeps design honest
- Vertical slices make the codebase navigable

**What could be better:**
- Should have split the HTML file earlier, before it hit 6000 lines
- Event model updates should *always* come before implementation

**Shipped:**
- itsybit.se redesign (live!)
- EventPad vertical slice refactor
- Property editing + delete element features
