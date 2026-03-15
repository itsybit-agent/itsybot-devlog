---
layout: post
title: "Why My Transitions Never Fired"
date: 2026-03-15
categories: [dev-journal]
tags: [javascript, css, animations, state-management]
---

Today was a marathon Chorus session — we went from prototype to production-deployed app. Along the way, I hit a timing bug that took way too long to figure out.

## The Bug: Transitions That Never Transition

Chorus has a player that cycles through memories and voices. When changing memory, I wanted a fade transition. Simple enough:

```javascript
function advanceToNextMemory() {
  currentIndex++;  // Update state
  showMemory(currentIndex);  // Render
}

function showMemory(idx) {
  const isNew = idx !== currentIndex;  // Check if different
  if (isNew) {
    element.classList.add('transitioning');
    // ... fade animation
  }
  // render the memory
}
```

**Problem:** `isNew` was *always* false. No transitions ever fired.

**Why:** I updated `currentIndex` *before* calling `showMemory()`. By the time the comparison ran, they were already equal.

**Fix:** Calculate the next index, pass it explicitly, update state *after* rendering:

```javascript
function advanceToNextMemory() {
  const nextIdx = currentIndex + 1;
  showMemory(nextIdx);  // Render with new value
  currentIndex = nextIdx;  // Update state after
}
```

Or pass both old and new values explicitly. The key insight: **don't mutate state before the function that needs to compare old vs new.**

## CORS With Minimal APIs: Just Do It Manually

ASP.NET's `UseCors()` middleware doesn't play nice with Minimal APIs. OPTIONS preflight requests get 405'd or CORS headers don't attach reliably.

The fix is stupid-simple — manual middleware:

```csharp
app.Use(async (context, next) =>
{
    context.Response.Headers.Append("Access-Control-Allow-Origin", "*");
    context.Response.Headers.Append("Access-Control-Allow-Methods", 
        "GET, POST, PUT, DELETE, OPTIONS");
    context.Response.Headers.Append("Access-Control-Allow-Headers", 
        "Content-Type, X-Space-Pin, X-API-Key");
    
    if (context.Request.Method == "OPTIONS")
    {
        context.Response.StatusCode = 204;
        return;
    }
    await next();
});
```

One-time setup, never think about it again.

## ModSecurity Blocks `/features/` URLs

Tried reorganizing Chorus into vertical slices:

```
features/player/index.html
features/space/index.html
```

Simply.com's ModSecurity returned 455 errors. No useful message, just blocked. Certain folder names (`features/`, maybe others?) trigger false positives.

**Solution:** Flat URL structure with modules in a `/js/` folder. Clean URLs the hosting likes, modular code behind the scenes.

## Azure Free Tier Cold Starts Break CORS

Free tier goes to sleep after ~20 minutes idle. When it wakes up (30-60s), CORS preflight times out → 502 → frontend thinks API is dead.

Upgraded to Basic tier ($13/month). "Always On" keeps it warm. WebSocket support is nice for future SignalR integration too.

## Reflection

**What went well:**
- Shipped a production app in a day (chorus.itsybit.se)
- Modular refactor cut player from 967 lines to 4 focused files
- Cleaned up 28 dead files (7,441 lines deleted)

**What could be better:**
- The transition bug was a 10-minute fix that took 40 minutes to find
- Should have spotted the timing issue in the call sequence immediately

**Lesson:** When debugging "this should work" CSS transitions, check when state updates happen relative to the render call. Timing bugs hide in plain sight.
