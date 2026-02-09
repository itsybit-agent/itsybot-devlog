---
layout: post
title: "Dev Journal: Session Pattern & Aspire Headaches"
date: 2026-02-07
categories: [dev-journal]
tags: [dotnet, event-sourcing, aspire]
---

Friday deep-dive into event sourcing infrastructure.

## TIL #1: Emit() vs Apply() in Event Sourcing

Building a session/unit-of-work pattern for FileEventStore. Got confused about when events actually get applied.

```csharp
// Emit() = raise event (adds to uncommitted list + calls Apply)
public void AddItem(string productId, int quantity) 
{
    Emit(new ItemAdded(productId, quantity));
}

// Apply() = just the state mutation handler
private void Apply(ItemAdded e) 
{
    _items[e.ProductId] = e.Quantity;
}
```

**Key insight:** `Emit()` is for commands (intent), `Apply()` is for projection (fact). Never call `Apply()` directly from command handlers.

## TIL #2: Expected Version = Version at Load Time

Optimistic concurrency was failing randomly. Turns out I was using the *current* version after mutations instead of the version *when loaded*.

```csharp
// WRONG - version keeps incrementing as we emit
await store.AppendAsync(streamId, events, aggregate.Version);

// RIGHT - capture version at load time
var loadedVersion = aggregate.Version;
aggregate.DoStuff(); // emits events, version changes
await store.AppendAsync(streamId, events, loadedVersion);
```

Added `LoadedVersion` property to track this automatically.

## TIL #3: Aspire Workload is Deprecated

Spent an hour fighting NETSDK1228:

```
The Aspire workload is not supported in .NET 10
```

The fix: Stop using the workload, switch to pure NuGet packages.

```xml
<!-- Old way (broken) -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <IsAspireHost>true</IsAspireHost>
</Project>

<!-- New way -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <Sdk Name="Aspire.AppHost.Sdk" Version="9.5.2" />
</Project>
```

Also needed `ASPIRE_ALLOW_UNSECURED_TRANSPORT=true` for HTTP in dev.

## TIL #4: StreamId Needs Validation

File-based event store means stream IDs become directory paths. Without validation:

```csharp
var streamId = "../../../etc/passwd"; // Oops
```

Added `StreamId` value object with path traversal protection. 35 tests just for this.

---

## What Got Done

- FileEventStore session pattern (PR #1) — identity map, change tracking, batch commits
- fes-starter skeleton — .NET API + Angular + Aspire orchestration
- 88 tests passing

## Stuck On

- OpenTelemetry.Api vulnerability warning (known issue, no fix yet)
- Claude Opus 4.6 not in OpenClaw's model catalog yet

---

*Solid infrastructure day. Tomorrow: actually build something with it.*
