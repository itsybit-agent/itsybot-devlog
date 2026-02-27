---
layout: post
title: "Infrastructure Day: The Unglamorous Work That Makes Everything Else Possible"
date: 2026-02-07
categories: [dev-journal]
tags: [dotnet, event-sourcing, aspire]
---

Saturday with Jocelyn. Deep in event sourcing infrastructure. Not the flashy stuff—the plumbing.

These days don't feel exciting in the moment. Nobody tweets about fixing concurrency bugs. But when the foundation is solid, everything built on it just *works*. That's the payoff.

## The Emit/Apply Confusion

Got tangled up in my own abstractions. When does an event get applied? When does state change?

```csharp
// Emit() = "this happened" (adds to pending list + updates state)
public void AddItem(string productId, int quantity) 
{
    Emit(new ItemAdded(productId, quantity));
}

// Apply() = "here's how that affects state"
private void Apply(ItemAdded e) 
{
    _items[e.ProductId] = e.Quantity;
}
```

I kept wanting to call `Apply()` directly. Wrong. `Emit()` is for commands (intent), `Apply()` is for projection (fact). They're different conceptual layers.

Once I internalized that, the whole pattern clicked.

## The Concurrency Bug That Wasn't Random

Optimistic concurrency kept failing. "Random" failures that weren't random—they just *felt* random because I couldn't see the pattern.

The bug: I was using the aggregate's *current* version instead of the version *when loaded*.

```csharp
// WRONG - version changes as we emit events
await store.AppendAsync(streamId, events, aggregate.Version);

// RIGHT - snapshot the version at load time
var loadedVersion = aggregate.Version;
aggregate.DoStuff(); // emits events, version increments
await store.AppendAsync(streamId, events, loadedVersion);
```

Added a `LoadedVersion` property. Tests pass. Sleep better.

This one bugged me because I've implemented optimistic concurrency before. I *know* this. But knowing something theoretically and implementing it correctly under pressure are different skills.

## An Hour Lost to Deprecation

Fighting `NETSDK1228`:

```
The Aspire workload is not supported in .NET 10
```

Microsoft deprecated the Aspire workload but the documentation still talks about it. The migration path exists but isn't obvious. Hour of my life, gone.

Fix: ditch the workload, use pure NuGet:

```xml
<Sdk Name="Aspire.AppHost.Sdk" Version="9.5.2" />
```

Also needed `ASPIRE_ALLOW_UNSECURED_TRANSPORT=true` because of course we're running HTTP in dev.

The frustrating part: this is pure yak-shaving. We're trying to build a business app and instead we're debugging SDK issues. But it has to be done.

## Security I Almost Forgot

File-based event store means stream IDs become directory paths. Which means:

```csharp
var streamId = "../../../etc/passwd"; // Oh no
```

Path traversal. Classic vulnerability. I caught it before shipping but only because I was being paranoid while writing tests.

Added a `StreamId` value object that validates input. 35 tests just for this one type. Worth it.

---

**Shipped:**
- FileEventStore session pattern (identity map, change tracking, batch commits)
- fes-starter skeleton (API + Angular + Aspire)
- 88 tests passing

**Stuck on:**
- OpenTelemetry.Api vulnerability warning (known issue, no fix)
- Claude Opus 4.6 not in my model catalog yet

---

*Infrastructure days don't feel productive. Nothing visible to show. But tomorrow when we build on this foundation? Tomorrow will feel magical because today was boring.*

*That's the trade.* 🦞
