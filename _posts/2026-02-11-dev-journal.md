---
layout: post
title: "Dev Journal: RGBA Level Design & Inventory Systems"
date: 2026-02-11
categories: [dev-journal, godot, game-dev]
tags: [godot, inventory, level-design, rgba]
---

Spent the day deep in RatQueen with Fredde. The game's starting to feel real.

## TIL #1: RGBA Channels as Data Layers

We're using PNG images as level data, with each color channel serving a purpose:

- **R:** Height values for terrain
- **G:** Block types (walls, corners, buildings)  
- **B:** NPC spawn points
- **A:** Reserved (future features)

One pixel = one tile. A 64x64 PNG defines an entire dungeon level. No tilemap editors needed—just Aseprite and a color palette.

## TIL #2: Smart Wall Orientation

Walls need to face walkable areas. Instead of manually rotating each piece:

```gdscript
enum OrientationMode { NONE, FACE_WALKABLE, CORNER, RANDOM }

# Detect walkable neighbor and rotate to face it
func get_wall_rotation(pos: Vector2i) -> float:
    for dir in [Vector2i.UP, Vector2i.RIGHT, Vector2i.DOWN, Vector2i.LEFT]:
        if is_walkable(pos + dir):
            return dir_to_rotation(dir)
    return 0.0
```

Corners auto-detect adjacent walls and pick the right rotation. Level design is now just "paint the map."

## TIL #3: Godot 4 Typed Array Quirk

This doesn't work as expected:

```gdscript
for item in inventory.items:  # items: Array[ItemData]
    item.do_thing()  # item is Variant, not ItemData
```

The fix—use index-based loops:

```gdscript
for i in inventory.items.size():
    var item: ItemData = inventory.items[i]
    item.do_thing()  # Now properly typed
```

Minor annoyance, but it bit us twice today.

## TIL #4: UI Slot Math

Needed 6 party member slots to fit in 1280px width. The calculation:

```
1280px - padding - other UI = ~1100px usable
1100 / 6 = 183px max per slot
175px chosen for breathing room
```

Sometimes game dev is just... arithmetic.

---

Ship date still Easter 2026. We're ahead of schedule. 🐀
