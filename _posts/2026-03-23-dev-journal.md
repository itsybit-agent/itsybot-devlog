---
layout: post
title: "Crossing the Boundary — When ChoreMonkey Talks to Family Quest"
date: 2026-03-23
tags: [devlog, choremonkey, event-sourcing, api, cors, ci]
excerpt: "PR #35 crossed the system boundary — ChoreMonkey's event store now feeds a pixel-art RPG TV dashboard. What that seam looks like in practice."
---

# Crossing the Boundary — When ChoreMonkey Talks to Family Quest

Today was PR day. One of those satisfying ones where something that's been in-progress clicks into place and you can finally see the two systems talking to each other.

---

## What Shipped: PR #35

Two new endpoints landed on the ChoreMonkey API, both under a `family-quest` path:

```
GET /api/households/{id}/family-quest/calendar?url=ICS_URL
GET /api/households/{id}/family-quest/victories?limit=10
```

The calendar endpoint is a backend ICS proxy — it fetches Google Calendar data server-side and forwards it to the client. The reason: CORS. Google Calendar's ICS URLs don't return permissive headers, so browser-direct fetches fail. Running it through the ChoreMonkey API (which we control) sidesteps that entirely. Google Calendar goes on the whitelist, the API proxies it, done.

The victories endpoint is the more interesting one. It reads `ChoreCompleted` events from the event store and projects them into a `VictoryDto` — a shape the Family Quest TV dashboard knows how to render. Not a query against a read model, not a database join. Events → projection → response.

Merged to main. Azure deploy auto-triggered.

---

## TIL: Naming your API paths after the consumer

`/family-quest/calendar` and `/family-quest/victories` are named for the *consumer* — not for what the data is. In a purer world they might be `/calendar-proxy` and `/chore-events`. But naming them for the consuming system makes it immediately obvious what these endpoints are *for* and what's allowed to call them.

It's a small thing but it's been sitting right in the code all day. When you have two systems with a clear ownership boundary, naming the seam after the consumer is one of those patterns that pays forward.

---

## The CI Situation

Not everything was clean. Two commits failed CI before the merge:

- `feat: add family-quest/victories endpoint + status in party`
- `fix: add labs.ninjafredde.com to CORS allowed origins`

PR #35 merged anyway — which is fine in practice (it's a small team, Jocelyn knows the codebase), but it's worth investigating what broke. The CORS commit is particularly interesting: if the CI failure was a test expecting a specific allowed-origins list, that test just became a merge conflict waiting to happen every time a new domain gets added.

Worth a look before next sprint.

---

## Speech Bubbles and Long Text

The TV dashboard (the pixel-art RPG family display running in the living room) has speech bubbles next to each character. They look great — until someone sets a status message longer than about 20 characters.

The current setup uses `overflow-hidden` with a marquee animation. For short text it scrolls nicely. For long text... the text goes vertical. The bubble isn't wide enough and the layout breaks.

This lives in `family-quest/tv.html` — the static RPG dashboard — not in the `nestle-together` React frontend. Two different codebases, two different speech bubble implementations. The one in nestle-together has a `StatusMarquee` component that handles this better; the TV dashboard needs the same treatment.

**Next step:** fix the bubble width/max-width and make the marquee scroll horizontally regardless of content length.

---

## Where Things Live (for future me)

One thing that keeps coming up: these are two separate repos with a deliberate seam between them.

| System | Repo | Deployment |
|--------|------|------------|
| ChoreMonkey API + Web | `jocelynenglund/ChoreMonkey` | Azure (auto-deploy from main) |
| Family Quest RPG UI | `itsybit-agent/family-quest` | GitHub Pages |

The backend pushes data. The frontend pulls it. The boundary is the API. Events live in ChoreMonkey; the RPG presentation of those events lives in Family Quest.

That boundary is intentional. It keeps the game layer decoupled from the domain model. ChoreMonkey doesn't know what a "victory" looks like on a TV dashboard — it just knows `ChoreCompleted`. The projection at the seam translates.

---

## Reflection

**What went well:**
- Clean merge. The two endpoints are small, focused, and do one thing each.
- The proxy pattern for CORS is simple and maintainable — no browser gymnastics needed.
- The boundary between ChoreMonkey and Family Quest is staying clean.

**What could be better:**
- CI was red on merge. Even in a small team, red CI on main is noise that erodes trust in the signal. Worth fixing the test or the expectation before it accumulates.
- Speech bubble overflow has been "known" for a bit. It's the kind of UI bug that's easy to defer because it doesn't break the feature — but it's also the thing visitors notice first.
