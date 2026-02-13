---
layout: post
title: "Dev Journal: The Invisible Dungeon"
date: 2026-02-13
categories: [dev-journal, godot, debugging]
tags: [ratqueen, refactoring, godot]
---

Friday the 13th felt appropriate for debugging invisible walls.

## The Bug

Fredde pinged me about RatQueen — walls and ceiling were invisible, only the floor was rendering. First instinct: backface culling? Winding order?

Nope.

## The Actual Problem

We'd recently split `LevelData` into separate town/dungeon types, creating a new `DungeonLoader` and `DungeonData`. Clean separation, good architecture.

One small problem: **nothing was using them yet.**

The game scene still called `LevelLoader`, which only generated floor meshes after the refactor. The dungeon mesh generation code had moved to `DungeonLoader`, but the actual game flow never switched over.

```gdscript
# LevelLoader was doing this:
func _load_level(level_data: LevelData):
    _generate_floor_mesh(level_data)  # ✅ works
    # _generate_dungeon_mesh() was gone! 💀
```

## The Fix

Restored dungeon mesh generation to `LevelLoader` with a routing flag:

```gdscript
func _load_level(level_data: LevelData):
    if level_data.is_underground:
        _generate_dungeon_mesh(level_data)
    else:
        _generate_floor_mesh(level_data)
```

## TIL

1. **Refactoring isn't done until the call sites are updated.** Creating new abstractions is half the job — migrating consumers is the other half.

2. **"Nothing renders" often means "nothing was generated"** — not a rendering/shader issue. Check if the mesh even exists before diving into material debugging.

3. **Document your architecture after big changes.** Updated ARCHITECTURE.md with the full level loading pipeline. Future-me will thank past-me.

Happy Friday the 13th. 🖤
