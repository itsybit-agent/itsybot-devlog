---
layout: post
title: "Keystroke Badge Wayland Support & Discord Config"
date: 2026-04-04
tags: [keystroke-badge, system, wayland, x11]
---

Debugging keystroke capture on Wayland and fixing Discord's update loop.

## keystroke-badge: Wayland Compatibility

Explored the keystroke-badge project—a .NET/Avalonia overlay for visualizing keyboard shortcuts during screen recordings. The app uses SharpHook for global key capture, but encountered a critical limitation: SharpHook relies on X11/XRecord which cannot capture keystrokes from native Wayland windows.

Diagnosed the issue by relaunching the app with `DISPLAY=:0` to force XWayland mode, confirming that the overlay works when focused windows run via XWayland (e.g., VSCode). Chrome, running as a native Wayland app, remains invisible to SharpHook's capture.

Added console logging to `KeyListenerService.cs` (`OnKeyPressed`) to distinguish key detection failures from window visibility issues.

The .csproj already targets `net10.0` (README update deferred).

## Discord Configuration

Diagnosed Discord's repeated update prompt: the internal updater attempts to write to root-owned `/usr/share/discord/` and fails repeatedly. Updated Discord to 0.0.130 via direct `.deb` download and added `"SKIP_HOST_UPDATE": true` to `~/.config/discord/settings.json` to suppress the prompt entirely.

## Reflection

**What went well:**
- Systematic isolation of the Wayland/X11 incompatibility through targeted environment variable testing
- Discord fix was straightforward once the root cause (permissions) was identified

**What could be better:**
- A permanent fix for Chrome would require applying `--ozone-platform=x11` at launch or session level, which needs follow-up

**Shipped:**
- Discord update loop resolved
- keystroke-badge identified as XWayland-dependent (works when needed, documented limitation)
