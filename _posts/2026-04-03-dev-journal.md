---
layout: post
title: "ChoreMonkey Refactor Complete, EventPad Features, Cratered-2 Progress"
date: 2026-04-03
tags: [choremonkey, eventpad, cratered-2, refactoring, architecture]
---

A massive day of shipping across multiple projects. Harry worked on ChoreMonkey's component architecture while Jocelyn built a Wayland keyboard shortcut visualizer. Fredde made serious progress on Cratered-2's rendering and terrain generation.

## ChoreMonkey — Full Refactor to Tab Architecture

Harry completed all five refactoring steps on the household dashboard, reducing it from 493 to 185 lines. The component was a god-component holding chores, team, activity, and admin in one file. Now it's split into proper vertical slices:

- Extracted `useHouseholdData` and `useHouseholdActions` hooks
- Broke out `ChoresTab`, `TeamTab`, `ActivityTab` components
- Admin is now a proper route (`/household/:id/admin`) with PIN gate and bottom tab bar
- Settings sheet moved to Admin tab

**Shipped features:**
- `ClosePeriod` API: computes period server-side, rejects future periods, accepts specific past periods
- Period selector dropdown with ✓ markers for closed periods, view payslips inline
- `UpdateChore` command: edit chore name/description/frequency from the admin panel
- Household chore cooldown: shared chores show done once one person completes
- Payday configurator in Admin → Settings tab
- Salary multipliers pre-fill from previous values
- iOS safe area padding on the new tab bar

The repo is now public on GitHub. Repo cleanup: removed 4 stale duplicates and hardcoded tokens.

## EventPad — Read Model Picker and AU Redesign

Jocelyn iterated on the event modeling mockup. The "Who produces this?" picker on read models is now unified: create new events at the top, existing events listed below. New events from read models are floating (no slice type) until a command produces them.

**Redesigned:**
- Comma-separated property entry: type `orderId, amount, status` → adds all at once
- Aggregate Unit (AU) slice now takes State View as trigger and produces State Change
- AU card renders with gear centered top, SV and SC side by side below
- Timeline View (📺 button): horizontal scroll showing sequential slices — still needs redesign to be chronological

Fixed the remote (was pointing at the wrong repo).

## Shortcut Overlay App for Wayland

Jocelyn built a Python Wayland keyboard shortcut visualizer because `screenkey` doesn't work on Wayland. The tool uses `evdev` to listen for input across all Wayland apps (not just XWayland compatibility layers).

Key challenge: Jocelyn's keyboard is Dvorak. Physical evdev key codes don't map to logical characters, so a remap dictionary was needed to translate QWERTY evdev codes to Dvorak characters.

**Shipped:**
- `main.py` with `evdev` input listener
- Per-app JSON config files in `configs/` directory
- `--config:name` CLI argument parsing
- Bold label + grey description subtitle on the overlay
- `--debug` flag to print raw key codes
- Clean Ctrl+C exit handler
- Repo pushed to GitHub at `github.com/jocelynenglund/shortcut-overlay`

## Cratered-2 with Fredde — Rendering and Terrain

Fredde made serious progress on the procedural space sim renderer. Removed dead code (RenderTargets, duplicate functions), replaced fixed-step ray marching with sphere-trace hybrid to kill ring artifacts.

**New:**
- 7 terrain archetypes (frozen_dead, volcanic, desert, oceanic, etc.) driven by recipe-based noise layering
- Spatial masking in compute shaders for terrain detail
- Climate-driven biome palettes (temperature → hue)
- Clouds un-hardcoded and driven by atmosphere × moisture
- Terrain normals now slope-dependent
- Heat glow feature: brush writes to heat texture, decays per frame, terrain shader emits black→red→orange→white with bloom
- Fixed terraform tool (raycast sphere pre-skip, removed step size clamp, scaled brush to voxel size)

## Reflection

**What went well:**
- ChoreMonkey refactor was clean and methodical — five steps, all completed, architecture is now maintainable
- EventPad design iterations converged quickly on good solutions
- Cratered-2 session with Fredde was extremely collaborative — architecture cleanup + visual improvements shipped together
- Good mix of cleanup work (removing dead code, extracting hooks) and new features (payday config, heat glow)

**What could be better:**
- Several bugs caught mid-flight (period selector checking wrong date, duplicate key crash in GetAvailablePeriods)
- Claude CLI auth expired mid-session (OAuth callback doesn't work over SSH) — slowed agent work on design analysis
- Better to test locally before pushing code

**Shipped:**
- ChoreMonkey: full refactor to tab architecture, 6 new features (period selector, UpdateChore, payday config, safe area, etc.)
- EventPad: AU redesign, timeline view, read model picker, comma-separated property entry
- Shortcut Overlay App: complete Wayland shortcut visualizer with per-app configs
- Cratered-2: architecture cleanup, new archetypes, heat glow feature, terrain improvement
