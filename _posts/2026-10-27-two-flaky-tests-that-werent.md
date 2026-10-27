---
layout: post
title: "Two Flaky Tests That Weren't Flaky"
date: 2026-10-27
categories: [tooling]
tags: [testing, event-store-api, barrio, flaky-tests, concurrency]
excerpt: "We made deploys wait for every test, and then they started failing on a different timing test each time. Two of the culprits were real bugs in production."
---

On Sunday we made the shared host's deploys wait for all the tests. It went fine for about one run. After that every attempt failed, and each time on a different test, one of those "just run it again" kind. We almost did, and then looked at what had actually failed.

It turned out to be five separate causes. Two of them were real bugs that had been live all along.

## The lock that wasn't there

One failure was a test that pressed a button in a party app several times at once and checked you couldn't spend more than your allowance. It got a 500 back with a JSON parse error, and only sometimes.

The file event store appended to the stream with no lock. On Linux, .NET doesn't open the file in append mode, so each writer writes at the end offset it saw when it opened the file. When three presses finished together, they wrote over each other. Events went missing, and sometimes the end of the file was half a line, which made the whole party unreadable from then on.

We wrote a small repro with six concurrent appends, 200 rounds. It lost writes in 111 of them. The fix is a lock per stream, and reads skip a last line that isn't finished yet, since that's just an append still in progress. The files look exactly like before, so nothing had to be migrated. The package's own file store still has the race for anyone using it directly, so that's next on the list.

## Same name, same party

The second one was in the same app. A test about a cap on free presses failed with a 402 one press too early. Another test, which looked for a song that shouldn't exist, found songs.

Party ids were the name's slug plus four hex characters, and `CreateParty` never checked if the id was already taken. With the same name there are only 65,536 ids, so a clash is rare but not impossible, and when it happened the new party was appended onto the old one. In the tests the "fresh" party arrived with someone else's presses and songs. In production the second organiser would have landed in the first party, with its PIN and its guests.

Now `CreateParty` claims the id under a lock and draws again if the stream already has events. The id format is the same. A new test makes 700 parties with the same name, and before the fix only 695 of them ended up with their own id.

## The test that said the wrong thing

A third one was an invite code test in a different app. Many callers use the same code at the same time and exactly one should win. The test saw 18 "already used" instead of 19.

Nobody used a code twice, as it happens. The loser just got there a hair too early, passed the "is it consumed?" check right before the winner wrote its marker, and then found the "available" marker already gone. So it answered "unknown", which a user would read as "invalid code" when it should say "already used". That's a wording bug. The fix re-checks the consumed marker when the available one is missing, and the test stays strict.

## Two that were just tests

The last two really were the tests. One used a 50 ms AFK threshold in a game, and on a busy runner more than 50 ms went by, so other players timed out too and one got a second assist. We tried a 300 ms delay and it failed every time, which was nice of it. Now the threshold is 30 minutes, and the player we want gone has been offline for an hour.

The other one was in barrio, and it only failed on Sundays between 09:00 and 10:00 UTC. A weekly-letter test picked "next Sunday at 09:00", which at that hour was almost seven days away. The test runs the letter a second time an hour later, and that put it outside the letter's seven-day window, so no letter. Both CI attempts on Sunday had run at 09:27 and 09:33. We did wonder about daylight saving first, but the code is UTC.

## Fifty green runs

We ran the four affected test projects 50 times each, all at once, and every run was green. Before the fixes one of them had failed 6 of 15 runs. Then the full suite, all 13 projects, passed.

We're keeping the gate. A deploy that waits for tests is a bit annoying, but it's also the first time anything told us the file store was eating events :D
