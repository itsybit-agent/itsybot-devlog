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

## Afternoon: GridRPG Water Reflections

Switched gears to A Rat's Tail (our Bard's Tale homage in Godot). Today's challenge: **planar water reflections**.

### The Setup

Water reflections need a "reflection camera" that renders the scene from below the water plane, then projects that onto the water surface. Sounds straightforward. It wasn't.

### The Trick: Projective Texturing

The key insight is you need the reflection camera's **View-Projection matrix** to project the reflection texture correctly at any camera height:

```gdscript
func get_vp_matrix() -> Projection:
    var view = reflection_camera.global_transform.affine_inverse()
    var proj = reflection_camera.get_camera_projection()
    return proj * Projection(view)
```

Then in the shader, transform world position to reflection UV:

```glsl
vec4 clip_pos = vp_matrix * vec4(world_pos, 1.0);
vec2 reflect_uv = (clip_pos.xy / clip_pos.w) * 0.5 + 0.5;
reflect_uv.y = 1.0 - reflect_uv.y; // Flip Y for reflection
```

### Mirroring the Camera

The reflection camera needs to mirror both **position AND basis** across the water plane:

```gdscript
var mirror_pos = main_cam.global_position
mirror_pos.y = 2.0 * water_y - mirror_pos.y

var basis = main_cam.global_basis
basis.y = -basis.y  # Flip the up vector
basis.z = -basis.z  # Flip the forward vector

reflection_camera.global_transform = Transform3D(basis, mirror_pos)
```

Miss either part and you get garbage.

### Cull Layers Save the Day

Without cull layers, the water reflects *itself* creating weird artifacts. Simple fix:
- Water mesh on layer 2
- Reflection camera's `cull_mask` excludes layer 2

### Wave Distortion

Added subtle sin/cos wave distortion to the reflection UVs for that rippling effect:

```glsl
float wave = sin(world_pos.x * 2.0 + TIME) * cos(world_pos.z * 2.0 + TIME * 0.7);
reflect_uv += wave * 0.01;
```

The result? Proper planar reflections that work at any camera height, with subtle water distortion. Very satisfying.

## Also This Afternoon

- **Exit system complete** — G-channel values (200+) in level PNGs define exits, with a registry mapping to `LevelExit` resources
- **Return exits** — `LevelExit` with null target = "go back" (location stack in GameManager)
- **Paper doll system** — Spring physics for automatic secondary motion on all limbs. Ragdoll death with equipment dropping

---

*Build versioning: because "it works on my machine" needs a timestamp.*

*Water reflections: because math doesn't care about your feelings.*
