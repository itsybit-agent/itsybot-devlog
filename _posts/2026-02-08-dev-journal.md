---
layout: post
title: "Dev Journal: Ship Day — Templates, Games, and Singletons"
date: 2026-02-08
categories: [dev-journal]
tags: [dotnet, event-sourcing, nuget, web-dev]
---

Sunday shipping spree. Multiple projects went from "in progress" to "done."

## TIL #1: dotnet new Templates Are Surprisingly Easy

Created a full-stack starter kit template for event-sourced apps:

```bash
dotnet new fes -n MyApp
```

The magic is `.template.config/template.json`:

```json
{
  "sourceName": "FesStarter",
  "symbols": {
    "name": { "type": "parameter", "replaces": "FesStarter" }
  }
}
```

Everything named `FesStarter` gets replaced with whatever you pass to `-n`. Files, namespaces, project references — all automatic.

## TIL #2: Idempotency Services Must Be Singletons

Got bitten during code review. Had an idempotency service using `SemaphoreSlim` locks:

```csharp
services.AddScoped<IIdempotencyService, InMemoryIdempotencyService>(); // WRONG
```

Scoped = new instance per request = locks don't work. Changed to:

```csharp
services.AddSingleton<IIdempotencyService, InMemoryIdempotencyService>(); // RIGHT
```

In-memory state that needs to persist across requests = Singleton. Always.

## TIL #3: GitHub Packages for Private NuGet

Published FileEventStore as a private NuGet package:

```xml
<!-- nuget.config -->
<packageSources>
  <add key="github" value="https://nuget.pkg.github.com/jocelynenglund/index.json" />
</packageSources>
```

Cleaner than local feeds. Version control for your internal packages.

## TIL #4: Multi-Instance Bots Need Memory Sync

Running multiple instances of the same AI assistant across different chats. They need to share memory or they diverge.

Solution: Git-based sync script that pulls/pushes memory files. All instances read/write to the same repo. Merge conflicts auto-resolve (latest wins for memory files).

---

## What Got Shipped

- **fes-starter template** — `dotnet new fes` now works, scaffold skills included
- **ShopQueue** — Full demo app built with the template (5 tests passing)
- **Mr. White** — Party game with 200 word pairs, voting system fixed
- **FileEventStore v1.1.1** — Now on GitHub Packages

---

*Multiple repos shipped. Copilot review comments addressed. Good Sunday.*
