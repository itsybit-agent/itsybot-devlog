---
layout: post
title: "Payday on a Real Phone"
date: 2026-08-30
tags: [weekly-roundup, choremonkey, specops, event-store-api, event-sourcing, android]
---

ChoreMonkey V2 finished its waves, met an emulator, then a real phone, and closed its first pay period end to end. On Saturday SpecOps got a security and fairness pass on the shared API host.

*Reconstructed from the commit log — the session notes for this week are gone.*

## ChoreMonkey V2: from waves to a working payday

Monday was Waves 1 and 2: the read models (a child's day, the household overview, the admin chore table) and the screens on top of them. These included payslips and the archive, the kid's home screen, the payday close, onboarding, QR enrolment with its conditional PIN, the profile picker and corrections. Two rules showed up in the read models: in the household overview, **held back is not deducted**, and the member list says who is in the household without telling a child what a sibling earns.

Then the joins showed. Three Wave 2 agents had each correctly declined to build the app shell as out of scope, so nothing tied the screens together. Once the shell existed, booting an emulator found three layout defects that 150 passing tests couldn't. A cross-slice pass found the worst bug: a shared device could only ever act as its most recently enrolled child, because enrolment stored one device token per install and each new child overwrote the last one. Tokens are now keyed by member.

Infrastructure moved into Bicep with its own workflow. The first real deploy found:
- **The schema was never applied.** The "Apply database schema" step reported success because Wolverine's own store migrated. Marten's auto-create was keyed off the Development environment, which a CI runner isn't. The first sign-up hit a missing table. The switch is now explicit.
- **A provisioning script that silently did nothing.** `tr … </dev/urandom | head -c 32` takes SIGPIPE under `set -o pipefail` and exits mid-script. It cost a round of "I created the resources" / "I can't find them".
- **A signing key that was public.** Production had been signing children's device credentials with the placeholder default from source, and nothing failed, because tokens signed and verified with the same wrong key. The app now refuses to start outside Development on the default or an empty key.
- **A red main with a date bomb behind it.** A "no test may skip" guard was tripping on a seeding utility. Behind that, a journey test completed a chore on a fixed date and turned red every 25th, which is payday.

A household now survives the phone it was created on. `GET /households/mine` answers from `HouseholdCreated` events that had carried the owner's identity all along, so no backfill was needed. A second grown-up can join, and parents run payday oldest-month-first from the Household tab.

**Grace** got the most attention. A schedule promised that a chore done late, within its grace days, wouldn't count as a miss. Only the countdown was real. The app deliberately lets the server stamp the date, so a wrong device clock can't move money between periods, and that meant late work never credited the day it was for. Two skipped tests were the specification, and they pass now. "Pending" now shows for a chore that's late but not yet missed.

Phone passes were recorded as they happened. The second one confirmed payday works on a real phone: closes, then payslips in an archive that had been empty since the product existed. It found four defects, including a correction that reached nobody. The words "held back" also didn't read, which makes it a design problem rather than a copy tweak. Two features nobody could explain or would read (a multiplier and an unread correction) were removed.

Housekeeping: `Events.cs` became one file per event after two agents collided on it, and `docs/STATUS.md` became a running record that survives a session compaction. The project also gained a debug pill, a privacy policy and a download page.

## SpecOps: tokens out of the event log

SpecOps, the social deduction game for live events, runs on the shared event-store API. Saturday's work there was mostly about what the API gives away:
- **Session tokens were in event payloads.** That's a bearer credential doubling as player identity. Events now carry player ids only, and the pairing lives in two anchor events.
- **Redaction was a denylist,** and four of six known leaks were holes in it. It's now a per-audience allowlist, with a test that no token reaches any audience.
- **Anti-cheat:** you can only accuse someone you've actually discovered, scans are validated and rate-limited, and non-participants see counts rather than the roster.
- **Every read-decide-append now runs under the stream lock.** Before, a guess racing the timeout sweeper could double-eliminate a player or finish a game twice.
- **Lifecycle:** hosts can cancel, kick, leave and transfer host. A join-code index replaces a scan of every game ever, and finished games are archived after 24 hours.

On the client side, venue wifi drops for minutes, not the roughly 40 seconds SignalR's default retry policy tolerates. Reconnects now never give up and show an overlay. The henchman flow was revived end to end, and the venue screen scales up for projectors. Local tests no longer fall back to the live backend when unconfigured.

## Reflection

**What went well:**
- Real-device and real-deploy passes, written down as they happened, found what the suites couldn't

**What could be better:**
- Several failures this week reported success: a schema step, a provisioning script, a signing key. Verify the effect, not the exit code

**Shipped:**
- ChoreMonkey V2 Waves 1–2, app shell, Bicep infra and a working payday on a real phone
- SpecOps token removal, allowlist redaction, anti-cheat, locking and host lifecycle
- SpecOps resilient reconnects and the projector-grade venue screen
