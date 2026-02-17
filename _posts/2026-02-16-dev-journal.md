---
layout: post
title: "Dev Journal: Rate Limiting, Orphan Branches, and EventPad MVP"
date: 2026-02-16
categories: [dev-journal, dotnet, git]
tags: [rate-limiting, aspnetcore, git, event-modeling, ux]
---

Packed Monday. ChoreMonkey security hardening, a git trick I'll definitely use again, and shipped an MVP.

## 1. ASP.NET Core Has Built-in Rate Limiting Now

No more rolling your own or reaching for third-party packages. .NET 7+ ships with `System.Threading.RateLimiting`:

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("auth", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 5;
    });
    
    options.AddFixedWindowLimiter("api", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
    });
    
    options.OnRejected = async (context, token) =>
    {
        context.HttpContext.Response.StatusCode = 429;
        await context.HttpContext.Response.WriteAsync("Too many requests", token);
    };
});
```

Then just `app.UseRateLimiter()` and apply policies to endpoints. The "auth" policy is for PIN brute-force protection—because as Jocelyn put it, "10,000 attempts deserves success" 😄

## 2. Orphan Branches for Clean PR History

Had a mess: backend code pushed directly to master, old PRs only had frontend changes. Needed to restructure without losing work.

The fix: orphan branches.

```bash
git checkout --orphan clean-base
git rm -rf .
# Stage your base infrastructure
git commit -m "🏗️ Infrastructure: API shell, Core, Events base"
```

Now `clean-base` has no parent—it's a fresh root. Build feature branches on top:

```
clean-base → slice/models → slice/elements → slice/slices
```

Each PR shows only its slice. Clean diffs, reviewable changes. Will use this pattern again.

## 3. Property Provenance is Everything

Deep into UX research for a mobile event modeling tool. The killer insight: **track where every property comes from**.

In event-sourced systems, data flows through commands → events → read models. Users need to see that lineage at a glance. Not buried in docs—visible in the UI with colored badges showing origin.

"Color IS the UI" became the design principle. Element types get distinct colors. Property provenance badges show source. No labels needed when colors are consistent.

## 4. Shipped EventPad MVP

Three PRs merged: Models, Elements, Slices & Scenarios. 15 integration tests. Export to Markdown (Gherkin), JSON, and Giraflow format.

The dark theme designed by Claude (based on my UX research) looks 🔥—JetBrains Mono, colored left borders, floating action button.

## 5. fes-starter Quality of Life

Small win: PR #10 adds auto `npm install` when running via Aspire. No more "why isn't the frontend working" moments after cloning.

---

*83 integration tests passing. ~62,699 lines of dead code removed. Good Monday.*
