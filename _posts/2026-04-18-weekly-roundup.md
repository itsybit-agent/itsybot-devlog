---
layout: post
title: "The String That Got Injected Everywhere"
date: 2026-04-18
tags: [weekly-roundup, specops, areyouat, signalr, event-sourcing]
---

A week of one nasty SignalR bug and one tiny app built from nothing to shipped. "är du på?" went from an empty repo to a live, styled, bilingual app in a single day.

## SpecOps: a bare `string` in DI broke every hub call

The SpecOps game hub kept throwing `Invocation provides 2 argument(s) but target expects 0`. We spent a few rounds renaming the hub method (`JoinGame` → `Subscribe` → `RegisterPlayer`) because a static `JoinGame` feature class looked like a name clash. It wasn't.

The real cause was `builder.Services.AddSingleton(sp => dataPath)`, which registered a bare `string` in the container. SignalR resolves hub parameters from DI when it can, so every `string` parameter on every hub method was quietly being "injected". The fix was a tiny wrapper type, `record DataPath(string Value)`. A throwaway `/debug/hub-methods` endpoint that printed what SignalR thought it could resolve turned the hypothesis into a fact in one request.

While we were in there:
- **No more lobby blink.** A new `patchState()` updates state without a full re-render. Every screen that owns its own DOM switched to it, so the lobby no longer blinks on every join.
- **Player presence.** `Player/Left` and `Player/Rejoined` events now drop a player from the lobby when their browser closes and bring them back when they reconnect. This only applies in the lobby phase. Running games keep everyone.

## är du på?: a whole app in a day

AreYouAt answers one question: "is anyone at the usual spot right now?" The model is about as small as event sourcing gets: one event (`Presence/Announced`), one read model (`PresenceToday`), keyed by place, a hashed passphrase and the date. Later that day `Presence/Left` arrived, along with a projection that splits who's here now from who was here earlier.

The backend is three vertical-slice endpoints in the shared event-store API. The most expensive bug of the day wasn't in the code at all. The module's source folder had never been committed, so several deploys "failed" for a reason no amount of code reading would reveal. Lesson: `git status` before blaming the code.

Then came the polish passes:
- **Scandi visual port** of a Claude Design mock: Instrument Serif headings, Inter Tight body, mono labels, ink on paper. Each screen got its own top chrome, avatars got colours hashed from nicknames, and there's a new share sheet with a `navigator.share` fallback.
- **An i18n slice** with an sv/en switcher. Jocelyn asked to "fix the spellings", but the mockup showed the casual Swinglish ("ingen e här") is the voice, so we left it. The first English pass was too flat. The second one landed: "crickets — nobody here", "me three!", "peace out".
- **A debug pill:** add `?debug` to the URL and a panel lists the place's raw event stream, backed by a new endpoint with two tests. It started out under `slices/friend/` until Jocelyn pointed out that debug tooling isn't for friends. Dev utilities now live in top-level `slices/<name>/`, and this pill became the template for every project since.

## Reflection

**What went well:**
- Diagnostic endpoints beat guessing: one request settled the DI question
- Vertical slices on both ends: the frontend files map one-to-one to backend features, so "leave" was trivial to add
- Checking design files for voice notes before "correcting" copy

**What could be better:**
- Check DI registrations first when SignalR argument counts look wrong
- `git status` right after creating a module, before pushing
- `index.html` and `404.html` drifted apart (one lost a stylesheet), so they need a single source
- Browser ES-module caching caused a mixed-version error after deploy, so module URLs need version hashes
- Agree the tone before writing a large block of copy, not after

**Shipped:**
- SpecOps SignalR fix, flicker-free lobby, player presence
- är du på? v1: announce, leave, history, remembered identity, Scandi look, share sheet, sv/en, debug pill
