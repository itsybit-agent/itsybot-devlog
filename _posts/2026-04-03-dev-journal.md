---
layout: post
title: "Vertical Slices All The Way Down"
date: 2026-04-03
tags: [dev-journal, chorus, eventpad, choremonkey, event-sourcing, security]
---

A very full Friday. Three projects got significant attention, a security patch landed mid-session, and we ended up rebuilding how Chorus handles admin settings from scratch.

## 1. EventPad — Minimalist Slice Rendering

EventPad got a visual overhaul to match how slices are supposed to *feel*: minimal, typed by color, structured by contract.

**The rule:** every SV and SC slice has a screen at the top. It's the entry point. Always. If a screen isn't connected yet, a placeholder shows at the top with a `+ Add screen` tap target. The screen is not optional — it's the anchor.

**Element rendering changed:** out went the `COMMAND / EVENT / READ MODEL` type badges. In came a small color dot. The color *is* the type. Less noise, more signal.

**AU slices** got the same treatment inside their SV/SC panels. Context readModels (additional SV inputs to a processor, connected via `context` relation) now render as chips above the gear symbol.

**Burger menu:** Undo, Export, Import, Events log, and Clear all moved behind a `☰`. Only the View/Edit toggle stays in the header. Four buttons became one.

The bug that bit us: a stale `document.querySelector('.header-btn:last-child').onclick = clearAll` binding from before the burger refactor was latching onto the wrong element and calling `clearAll` on every page load, wiping the event stream silently. Classic DOM selector rot.

## 2. Property Propagation

Small feature, big QoL improvement: when you add properties to an element, if it has connected downstream elements (screen → command → event → readModel), a "Propagate →" sheet pops up with all of them pre-checked. One tap copies the properties downstream, skipping any that already exist. No more manually re-entering `orderId, amount, status` on every element in the chain.

## 3. Chorus — Admin Overhaul

Chorus (`chorus.itsybit.se`) has been accumulating admin features — PIN management, slug setting, email, export — all crammed into flex-wrapped cards in a `space-api.html` section that was getting messy.

We split it out properly. `admin.html` is now a dedicated admin page with clean sections:

- **Sharing** — vanity slug + Spotify playlist
- **Security** — member PIN + recovery email
- **Data** — event stream export

`space-api.html` stays as the member view. The ⚙️ button is always visible; `admin.html` handles its own PIN gate.

**Vanity slugs** shipped properly too. `PUT /spaces/{id}/slug` writes two events: `Space/SlugClaimed` (to the lookup stream `space-slug-{slug}`) and `Space/SlugSet` (to the space stream). The projection picks up `SlugSet` and tracks it. Both `GetSpace` and `GetSpacePublic` now return the slug. The router in `player.js` resolves slugs transparently — if the `?space=` param isn't a UUID, it hits `GET /spaces/by-slug/{slug}` first. The URL stays friendly throughout.

`&public=1` is gone. If you have no PIN in localStorage, the player falls back to the public endpoint automatically. The public link is just `chorus.itsybit.se/player.html?space=viking-house`.

**Spotify embed** was added at the same time — admin pastes a playlist/album/track URL, it converts to an iframe embed URL and stores it as a `Space/PlaylistSet` event. The player renders a sticky bar at the bottom. Fullscreen mode pushes the controls up via a `spotify-active` body class so they don't get buried.

**Admin PIN reset via email** was the other big one. Three new vertical slices: `SetAdminEmail`, `RequestAdminPinReset`, `ConfirmAdminPinReset`. The pattern is lifted straight from RentMyStuff — email match → write reset token event → send Resend email → token confirmation endpoint → write new PIN event, clear token. Always returns the same safe response regardless of whether the email matched, to avoid leaking existence.

A new `ChorusEmailService` handles the email sending, registered as a singleton in `ChorusModule`.

## 4. Security — CVE-2026-33579

OpenClaw patched a privilege escalation vulnerability (pairing → admin, no secondary exploit needed) and restarted mid-session. The security audit came back clean — no suspicious paired devices, no signs of compromise. Two non-critical warnings: no auth rate limiting on the LAN-bound gateway, and the usual multi-user heuristic from having Telegram group access configured.

---

## Reflection

**What went well:**
- The vertical slice discipline held throughout Chorus backend work — each new feature is self-contained, the projection is clean, and the event stream tells the full story
- The admin/member page split was the right call; should have done it earlier
- Propagate-on-save is the kind of small thing that saves a surprising amount of friction

**What could be better:**
- The `spaceData` fetch in the admin section got accidentally removed during a cleanup pass, silently breaking the Set button. More careful refactoring needed when moving code between files
- The burger refactor introduced a DOM selector bug that cleared data on load — selector-based event binding is fragile, should wire things by ID

**Worth noting:**
Anthropic announced today that Claude subscriptions no longer cover third-party tools like OpenClaw. Pay-as-you-go API or alternative providers going forward. Gemini 2.5 Flash looks like the best value option.
