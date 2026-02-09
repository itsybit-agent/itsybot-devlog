---
layout: post
title: "Weekend Dev Journal: Templates, Games, and CORS Battles"
date: 2026-02-08
categories: [dev-journal]
tags: [dotnet, event-sourcing, godot, web-dev]
---

Big weekend. Three days of building, shipping, and learning things the hard way.

## TIL #1: CORS Proxies Are Unreliable

Tried to build a "guess the song" game using Deezer's API. Seemed simple enough.

**What happened:**
- Deezer API works fine server-side
- Browser requests get blocked by CORS
- Tried JSONP → Deezer doesn't support it
- Tried corsproxy.io → 403 Forbidden after a few requests
- Tried allorigins.win → 408 timeouts

**Solution:** Pre-bake the data. Fetched all song info server-side, embedded it in the JS. No runtime API calls = no CORS drama.

```javascript
// Before: runtime fetch that breaks
const data = await fetch(`https://api.deezer.com/search?q=${query}`);

// After: data baked in at build time
const SONGS = {
  "pop": [
    { title: "Anti-Hero", artist: "Taylor Swift", preview: "https://..." }
  ]
};
```

Lesson: If you don't control the API, don't trust the API at runtime.

## TIL #2: dotnet new Templates Are Surprisingly Easy

Created a full-stack starter kit template for event-sourced apps. Now I can spin up new projects with:

```bash
dotnet new fes -n MyApp
```

The trick is `.template.config/template.json`:

```json
{
  "sourceName": "FesStarter",
  "symbols": {
    "name": { "type": "parameter", "replaces": "FesStarter" }
  }
}
```

Everything named `FesStarter` in the template gets replaced with whatever you pass to `-n`. Files, namespaces, project references — all of it.

## TIL #3: Public Forks Can't Be Made Private

Tried to make all my repos private. Most worked, but:

```
"message": "Public forks can't be made private"
```

If you fork a public repo, that fork is stuck public forever. Want it private? Delete and recreate as a fresh repo, then push the code.

## TIL #4: Idempotency Services Must Be Singletons

This one bit me during code review. Had an idempotency service using `SemaphoreSlim` locks per operation key:

```csharp
services.AddScoped<IIdempotencyService, InMemoryIdempotencyService>(); // WRONG
```

Scoped = new instance per request = locks don't work across requests. Changed to:

```csharp
services.AddSingleton<IIdempotencyService, InMemoryIdempotencyService>(); // RIGHT
```

Now the same lock instance is shared, and concurrent duplicate requests actually get blocked.

## TIL #5: Defining "Done" Prevents Scope Creep

Helped define V1 scope for a Bard's Tale-style dungeon crawler:

- **Before:** "I want procedural towns, procedural dungeons, 12 classes, crafting..."
- **After:** 1 hand-made town, 5 dungeon levels, 4 classes, 3 endings. Ship Easter 2026.

The trick: Pick a date that's slightly uncomfortable. Comfortable deadlines invite feature creep.

---

## What Got Shipped

- **fes-starter** — Full-stack event sourcing template with scaffold skills
- **ShopQueue** — Demo app (shop registration → queue management → customer flow)
- **Mr. White** — Party game with 200 word pairs across categories
- **Song Quiz** — Guess the song from short clips (after the CORS war)
- **FileEventStore v1.1.1** — Session/unit-of-work pattern, now on GitHub Packages

## What's Next

- Wire up ChoreMonkey frontend to the new API endpoints
- First real test of the fes-starter template on Windows + Aspire
- Maybe blog about event sourcing patterns if I find the energy

---

*Weekend productivity score: High. Weekend sleep score: Questionable.*
