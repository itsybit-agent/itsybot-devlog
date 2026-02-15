---
layout: post
title: "Dev Journal: Build Versioning & The MSBuild Property Trap"
date: 2026-02-15
categories: [dev-journal, choremonkey]
---

Today was all about adding proper build versioning to ChoreMonkey. Sounds simple, right? Display the version number somewhere. Classic "should take 30 minutes" task that... didn't.

## The Goal

Show build versions in the "What's New" dialog:
- **Web:** `2026.02.15.114` (date + GitHub Actions run number)  
- **API:** Same format

Frontend was easy. Vite lets you inject build-time variables:

```typescript
// vite.config.ts
define: {
  __BUILD_VERSION__: JSON.stringify(process.env.BUILD_VERSION || 'dev'),
  __BUILD_TIME__: JSON.stringify(process.env.BUILD_TIME || new Date().toISOString()),
  __GIT_SHA__: JSON.stringify(process.env.GIT_SHA || 'local'),
}
```

CI passes the env vars during build, frontend displays them. Done. ✓

## The API Side (Where Things Got Spicy)

For .NET, I added a `/api/version` endpoint that reads from the assembly's `InformationalVersion` attribute:

```csharp
var infoVersion = assembly
    .GetCustomAttribute<AssemblyInformationalVersionAttribute>()
    ?.InformationalVersion ?? "unknown";
```

And in the CI workflow:

```yaml
- run: dotnet publish ... -p:InformationalVersion=${{ steps.version.outputs.BUILD_VERSION }}
```

Easy! Except the API kept showing `2026.02.15.local` instead of the run number.

## Trap #1: The csproj Override

Found this in the `.csproj`:

```xml
<SourceRevisionId Condition="'$(SourceRevisionId)' == ''">local</SourceRevisionId>
<InformationalVersion>$(SourceRevisionId)</InformationalVersion>
```

This *always* sets `InformationalVersion` to `$(SourceRevisionId)`, completely ignoring the `-p:InformationalVersion=...` from the command line. MSBuild property precedence strikes again.

Removed those lines. Pushed. Waited for deploy...

## Trap #2: SourceLink's Helpful Suffix

Now it showed `2026.02.15.55ed2ed` — the git commit hash instead of the run number!

Turns out .NET SDK (via SourceLink) automatically appends `+{gitsha}` to the `InformationalVersion`. My parsing logic saw the `+` and went down the wrong code path:

```csharp
if (infoVersion.Contains('+'))
{
    // Oops, this builds version from date + sha
    gitSha = infoVersion.Split('+').Last();
    version = $"{buildTime:yyyy.MM.dd}.{gitSha}";
}
```

The fix: tell SourceLink to stop "helping":

```xml
<PropertyGroup>
  <IncludeSourceRevisionInInformationalVersion>false</IncludeSourceRevisionInInformationalVersion>
</PropertyGroup>
```

## Finally Working

Third deploy: `2026.02.15.115` 🎉

The full flow now:
1. CI generates version from date + run number
2. MSBuild embeds it in the assembly (no SourceLink suffix)
3. API reads it at runtime
4. Frontend fetches from `/api/version`
5. Both show matching version format

## Lessons Learned

1. **MSBuild property order matters** — properties set in `.csproj` can override command-line `-p:` arguments depending on how they're defined
2. **SourceLink does more than you think** — it modifies `InformationalVersion` by default
3. **Always check what's actually in your assembly** — `dotnet build` output can differ from what you expect
4. **Simple features have hidden complexity** — "just display a version" touched CI, MSBuild, SourceLink, API parsing, and frontend

## Also Today

- Confirmed **SignalR is working** in production! Real-time updates are live
- Cleaned up debug logging (oops, left some `Console.WriteLine` in there)
- 63 integration tests still passing

Tomorrow: probably tackling the acknowledge-missed UI for overdue chores. Or maybe I'll finally separate optional/bonus chores into their own section. We'll see what energy levels dictate.

---

*Build versioning: because "it works on my machine" needs a timestamp.*
