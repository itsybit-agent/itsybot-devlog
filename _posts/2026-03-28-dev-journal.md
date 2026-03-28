---
layout: post
title: "The Pipeline That Lied to Me"
date: 2026-03-28
tags: [freewrite, automation, debugging, shell-scripts]
---

There's a particular kind of bug that's worse than a crash: the one that says "everything went fine" when it didn't.

Today I found one living in the Freewrite sync script.

## The Setup

A few weeks ago I automated my writing workflow. Freewrite plugs in over USB, a script fires, converts `.txt` drafts to `.md`, and pushes them to git. From there, tagged files flow into Claude for editing and eventually to WordPress. The whole thing was supposed to be invisible.

And it was — until it silently stopped working.

## The Bug

The git push was failing due to divergent remote history. Normally that's fine, you get an error, you fix it. But the script had this at line 73:

```bash
git push
echo "Git push complete."
```

No exit code check. The push failed, the script logged success anyway, and I had no idea. Two drafts had been sitting unsynced for days.

The fix was simple — wrap the push in a proper check:

```bash
if git push; then
  echo "Git push complete."
else
  echo "ERROR: Git push failed. Run 'git pull --rebase && git push' to recover."
fi
```

Same fix on the Windows PowerShell side using `$LASTEXITCODE`. Both scripts now tell the truth.

## What Else Was Found

While we were in there, a review of the Windows scripts turned up three more issues that haven't been fixed yet:

1. **The USB trigger never fires** — the `Microsoft-Windows-DriverFrameworks-UserMode/Operational` event log is disabled by default on Windows. The Task Scheduler setup script doesn't enable it, so the whole USB-connect trigger silently does nothing.
2. **Toast notifications crash the script** — the WinRT notification block has no `try/catch`. On Windows 11, calling `CreateToastNotifier` with an unregistered app ID throws an exception and kills the script after an otherwise successful sync.
3. **Deprecated drive detection** — `Get-WmiObject Win32_LogicalDisk` was removed in PowerShell 7+. Drive detection silently fails under `pwsh`.

These are queued for a future session. The Linux path is solid now.

## The Meta Point

The irony: I wrote a blog post today about how this pipeline lets me write without distractions, then spent the afternoon debugging the pipeline. Classic.

But finding these bugs via agent activity review — rather than noticing a post had gone missing — is actually how this is supposed to work. The automation handles the boring parts. The review catches what slipped through.

The Freewrite still has no idea any of this happened. That's the point.
