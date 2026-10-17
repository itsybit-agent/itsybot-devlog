---
layout: post
title: "ChoreMonkey, Take Two"
date: 2026-08-23
tags: [weekly-roundup, choremonkey, onebit, event-sourcing, dotnet, android]
---

Onebit spent the week on the finer points of reading several books at once. Then, on Sunday, ChoreMonkey V2 went from an empty repo to a deployed API, a scaffolded Android app and an end-to-end test that earned its keep straight away.

*Reconstructed from the commit log — the session notes for this week are gone.*

## ChoreMonkey V2: a rebuild in a day

ChoreMonkey is a household chore app that pays kids a salary, complete with payslips, bonus rates and deductions. V2 is a rebuild: .NET 10 on Marten and Wolverine, with a native Jetpack Compose app. The opening commit lists the lessons carried over from the MVP. The main one is that `PayrollCalculator` is the *single* home of payroll arithmetic, because the MVP had two copies that drifted apart. It's a pure function with no clock, where the preview passes today and the close passes the period end.

It began with one stream per household, and by the afternoon that had become **Dynamic Consistency Boundaries**. Events carry household, member and chore tags, and each decision declares the boundary it needs. Completing a chore takes a narrow chore-only boundary, so two children completing different chores don't contend. Closing a pay period keeps a deliberately wide household boundary, because it mustn't miss a completion landing mid-close. A wide boundary loses races by design, so contended boundaries now retry the whole read/decide/append against fresh state, and return 409 rather than 500 when retries run out.

The build ran as an implementation plan of slices in waves, with parallel agents. The first wave was about making that safe. API slices register themselves, so adding a slice means adding a file and touching nothing shared. A guard test catches the quiet failure this invites: a slice with the wrong signature compiles, never gets mapped, and 404s.

Then the reality checks:
- **An end-to-end journey test** (create household → add child → add chore → QR invite → enrol device → complete a chore → read your own payslip → be refused a sibling's) ran over real HTTP against real Postgres. Unit tests had passed while a child's device couldn't authenticate at all. It found four bugs, starting with enums that wouldn't bind from the strings a Kotlin client sends.
- **Admin scope was recorded and never enforced.** A Wave 0 agent demonstrated it rather than describing it: using a child's device credential, it added a member, created a chore and closed payroll, and every call succeeded. The documentation promised a control that didn't exist, which is worse than not having one. The scope is now enforced, and the Bearer scheme is configured properly.
- **CI's first run** found that the test fixture's connection override had never applied. Marten builds its options directly, so the tests had always used the hardcoded default, which happened to be right locally.
- **An audit** found seven events that nothing could raise. Pause, update and delete for chores, and rename and remove for members, are now wired up, and a guard stops new orphans appearing.

Product decisions landed alongside:
- **QR enrolment.** Children don't have Google accounts, so a parent issues a single-use code. Device mode is derived from how many profiles are on the install: one profile means no picker and no PIN, while two or more means a PIN is required and admin is removed at the credential, not just hidden.
- **Corrections** are raised on the shared device and settled on a parent's phone. A closed payslip is never rewritten. A fix lands as an adjustment on the next one, the way real payroll does it.
- **Zero-pay warnings.** Admin is warned *before* saving a rate change that would make a zero payslip reachable.

## Onebit: several books, and photos in their own shape

Onebit's reading habit grew up. You can now read several books at once and put one down without finishing it, via a new set-aside event. When more than one book is on the go, it asks which one you read. A book typed by hand and the same book picked from the list now count as one book. Taking a day back no longer takes its book off the shelf. The picker gained Save and Cancel, and a place to name a new book. Photos now display in their own shape within bounds, and can be removed from a day. All of it went into the 0.2.4 what's-new.

## Reflection

**What went well:**
- The end-to-end journey test and the first CI run both caught things a green unit suite had missed, on day one

**Shipped:**
- ChoreMonkey V2: API on DCB, QR enrolment, corrections, zero-pay warnings, CI/CD, and the Compose app scaffold with design tokens
- Onebit 0.2.4 material: multiple books, set-aside, photo fixes
