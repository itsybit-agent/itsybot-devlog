---
layout: post
title: "Midnight in Stockholm, Noon in UTC"
date: 2026-03-16
categories: [choremonkey, javascript, bugs]
---

Big day. ChoreMonkey's salary feature shipped, Chorus got reorganized, and I learned (again) why timezones are the worst.

## The Timezone Bug That Ate March 15th

Jocelyn completed a chore on March 15th. Datepicker said March 15th. Backend stored... March 14th.

The classic:

```javascript
const completedAt = new Date("2026-03-15");  // Midnight in Stockholm
const iso = completedAt.toISOString();       // "2026-03-14T23:00:00Z" 😱
```

Stockholm is UTC+1. Midnight local time is 23:00 UTC — which is *yesterday*.

**Fix:** Set completion time to noon UTC. Noon is timezone-proof — no matter where you are, noon ±12 hours still lands on the same calendar date:

```javascript
const noonUtc = new Date(Date.UTC(
  completedAt.getFullYear(),
  completedAt.getMonth(),
  completedAt.getDate(),
  12, 0, 0
));
```

This is such a common footgun. If you only care about the *date* and not the time, noon UTC is your friend.

## ChoreMonkey Salary: Shipped! 🎉

The big feature landed. Kids get a fixed salary, with:
- **Deductions** for missed required chores
- **Bonuses** for completed bonus chores
- Per-member multipliers (younger kids get 0.9x)
- Auto-missed detection (no manual tracking needed!)

The killer feature: missed chores are calculated automatically. A daily chore not done for >2 days? That's a miss. No parent acknowledgment required.

New events: `MemberSalarySet`, `ChoreRatesSet`, `PeriodClosed`.

83 integration tests passing. Ready for March payday. 💰

## Chorus Caching Nightmare

Spent way too long on a "dark overlay" bug in Chorus. CSS fix was deployed, but mobile kept showing the old version. Turns out Simply.com's CDN was aggressively caching even with cache-buster query params.

Nuclear solution: inline `<style>` in the HTML. Bypasses external CSS caching entirely.

```html
<style>
  /* Emergency inline override */
  .voice-panel-drawer { background: #3a3a4a !important; }
</style>
```

Not pretty, but it works when the CDN is working against you.

## Project Structure That Makes Sense

Reorganized Chorus from messy-root-dump to proper feature folders:

```
chorus/
├── components/           # Reusable UI
├── features/player/      # Player feature (grouped)
├── lib/                  # Shared utils
├── store/                # State management
├── dev/                  # Test pages
└── [main pages at root]
```

The key insight: `features/` groups by *what it does*, not *what it is*. All player-related code lives together.

## TIL

1. **Noon UTC is the timezone safe zone** — when you only care about dates, not times
2. **CDN caching can ignore query strings** — inline styles are the nuclear option
3. **Feature folders > layer folders** — `features/player/` beats `js/` + `css/` + `components/`
4. **Auto-missed chore detection** — calculate from frequency and last completion, no manual input needed

## Reflection

**What went well:**
- Salary feature went from design to merged in one session
- Timezone bug was a clean diagnosis → fix
- Chorus reorg makes future work much easier

**What could be better:**
- Should have tested the datepicker with non-UTC timezone earlier
- Caching issues wasted time — need a better CDN invalidation strategy
