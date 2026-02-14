---
layout: post
title: "Dev Journal: Overdue Grace Periods & Personal Read Models"
date: 2026-02-14
categories: [dev-journal, dotnet, event-sourcing]
tags: [csharp, read-models, event-sourcing, choremonkey]
---

Back on ChoreMonkey today. Built out the "overdue chores" feature and learned some things about read model design along the way.

## TIL #1: Grace Periods Make UX Bearable

First pass at overdue detection: show everything you ever missed. Turns out that's... aggressive. Nobody wants to see "You missed vacuuming 47 days ago" when they open the app.

The fix: **grace periods**. Only surface recent misses:

- **Daily chores:** Only yesterday counts as overdue
- **Weekly chores:** Only last week (not 2+ weeks ago)
- **Interval chores:** One period back maximum

```csharp
// Daily: only check yesterday
var yesterday = today.AddDays(-1);
if (!WasCompletedOn(yesterday))
    overdueChores.Add(chore with { OverduePeriod = "yesterday" });

// Weekly: only check last week
var lastWeekStart = today.AddDays(-(int)today.DayOfWeek - 6);
var lastWeekEnd = lastWeekStart.AddDays(6);
if (!WasCompletedBetween(lastWeekStart, lastWeekEnd))
    overdueChores.Add(chore with { OverduePeriod = "last week" });
```

Older misses are silently forgiven. Fresh start energy.

## TIL #2: Personal vs Shared Read Models

Had a "My Chores" view showing what's assigned to the current user. First instinct: filter the existing chore list client-side.

Problem: the shared list doesn't have *your* completion status baked in. A chore assigned to "everyone" shows as complete if *anyone* finished it—not helpful when you need to know if *you* did it.

Solution: dedicated read model with personal context:

```csharp
app.MapGet("/api/households/{id}/my-chores", async (
    Guid id, 
    Guid memberId,
    IEventStore store) =>
{
    var events = await store.FetchEventsAsync($"household-{id}");
    
    // Build chore state with YOUR completion status
    var myPending = new List<MyChoreDto>();
    var myCompleted = new List<MyChoreDto>();
    var myOverdue = new List<MyOverdueChoreDto>();
    
    foreach (var chore in chores.Where(c => IsAssignedTo(c, memberId)))
    {
        var myStatus = GetCompletionStatus(chore, memberId, today);
        // Categorize based on personal status...
    }
    
    return new { pending = myPending, completed = myCompleted, overdue = myOverdue };
});
```

Key insight: **same events, different projections**. The admin sees "3/4 members completed this." You see "I still need to do this."

## TIL #3: Chores Can Be Both Pending AND Overdue

Edge case that wasn't obvious: a weekly chore can be:
- **Overdue** for last week (you missed it)
- **Pending** for this week (still due)

These aren't mutually exclusive states. The UI needs to show both—overdue as a nudge, pending as today's task. Same chore, two lists.

```csharp
// A weekly chore appears in overdue (last week) AND pending (this week)
if (missedLastWeek && !acknowledgedMiss)
    myOverdue.Add(BuildOverdueDto(chore, "last week"));
    
if (notCompletedThisWeek)
    myPending.Add(BuildPendingDto(chore));
```

## TIL #4: Acknowledge vs Complete

Users wanted a way to clear overdue items without lying about completing them. "I didn't walk the dog yesterday, but stop bugging me about it."

New event: `ChoreMissedAcknowledged`. It's not a completion—it's acceptance. The chore disappears from overdue without inflating your completion stats.

```csharp
public record ChoreMissedAcknowledged(
    Guid ChoreId,
    Guid MemberId,
    string Period  // "2026-02-13" or "2026-W06"
);
```

Honest UX > gamified UX.

---

57 tests passing. Ship it. 🐵
