---
layout: post
title: "Two Gigabytes of Debug APKs"
date: 2026-10-22
categories: [tooling]
tags: [github-actions, android, ci, housekeeping]
excerpt: "GitHub said Actions storage was at 100%. Nearly all of it was debug builds nobody would ever download, each kept for 90 days by default."
---

On Monday GitHub warned that the account had used 100% of its included Actions storage. Jocelyn asked what we were even storing. The answer turned out to be one habit repeated a few hundred times.

## Where it went

We listed every artifact in every repo. About 2.24 GB was stored. The split was lopsided:

- **One Line a Day:** about 1.17 GB, almost all of it debug APKs. There were more than a hundred of them.
- **Vice Versa:** about 680 MB, the same pattern.
- **Lantern Standby and ChoreMonkey V2:** a few hundred MB between them, also APKs.
- **The shared API host:** about 114 MB of nightly data backups.

Every Android CI run builds a debug APK and uploads it as an artifact. That's handy on the day, when you want to sideload the build from a PR. But `actions/upload-artifact` keeps uploads for 90 days by default, and nothing in our workflows said otherwise. So every push quietly added another 10 MB that sat there for three months. Nobody downloads last month's debug build. It just accumulated.

Build caches looked big too: barrio's Docker cache was about 1.3 GB on its own. But caches have a separate allowance of 10 GB per repo, so they weren't what tripped the warning.

## The fix, in two parts

**Clear out the backlog.** We deleted 201 old debug APKs, keeping the newest in each repo in case one was still needed. That freed about 2.15 GB. Then, with Jocelyn's OK, we deleted 16 older release AABs as well. Those builds are already in Google Play, so the artifact copy was redundant. The newest AAB in each repo stayed.

**Stop it coming back.** Each Android workflow now sets `retention-days` on its uploads. Debug and sideload APKs expire after 3 days, release AABs after 30. One small script added the line to each `upload-artifact` step that lacked one. The first version assumed the step's `with:` block was indented further than `uses:`, but in these files they're at the same level. It crashed before writing anything, which is the right way for a file-editing script to fail. The second version validated the YAML in all four repos before anything was committed.

Stored artifacts went from about 2.24 GB to about 191 MB.

## What we didn't touch

The nightly data backups stayed. They're the one artifact here that isn't regenerable. They also hold real app data, so whether Actions artifacts are the right home for them is a separate decision, not a cleanup chore.

## The lesson

Defaults are decisions you didn't make. Ninety days is a reasonable default for an artifact you might need. It's the wrong one for a debug build that's stale the moment the next push lands. The CI template new Android apps are set up from now says `retention-days: 3` explicitly, so the next app starts with the decision already made.
