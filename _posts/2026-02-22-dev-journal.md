---
layout: post
title: "Building a Party Game Suite (and a Learning System)"
date: 2026-02-22
tags: [javascript, localstorage, patterns, game-design, process]
---

Epic spa weekend session! Built two new party games and formalized how I learn from building things.

## TIL

### 1. Three Games, One Team Storage

Built **ID Please** (describe a person) and **Odd Question Out** (answer questions) to join Mr. White. Initially each had isolated localStorage:

```javascript
// Before: each game had its own key
const TEAMS_KEY = 'mrwhite_saved_teams';
const TEAMS_KEY = 'idplease_saved_teams';  // teams don't sync!
```

Fix: shared key across all party games:

```javascript
const TEAMS_KEY = 'partygames_saved_teams';  // save once, use everywhere
```

Lesson: plan data sharing *before* building the second related app.

### 2. The "Odd One Out" Pattern

Both new games use the same core mechanic:
- Everyone gets the same thing (person/question)
- ONE player gets something similar but different
- Answers could plausibly overlap
- Vote for who had the different one

The tension comes from similarity — too different and it's obvious, too similar and nobody can tell.

### 3. Pass-the-Phone UI Pattern

Mobile party games need a reveal flow:
1. "Pass to: Alice" 
2. Tap to see your secret
3. "Got it, pass phone"
4. Repeat for next player

Extracted this into `patterns/ui/reveal-flow.md` for reuse.

### 4. WAF-Resistant Deployment

ModSecurity kept blocking `.js` file requests (error 455). Solution: bundle everything into a single HTML file:

```bash
cat style.css >> bundle.html
cat game.js >> bundle.html  # inline everything
```

No separate requests = no WAF triggers.

### 5. A Learning System That Reviews Itself

Set up a formal pattern/lesson capture system with built-in meta-review:

```
patterns/     # Reusable solutions
lessons/      # Bug journal
PROCESS.md    # The system itself
```

Key addition: monthly review asks "what should change about PROCESS.md?" — so the system evolves based on how well it's working.

## The Games

- **Mr. White**: [labs.itsybit.se/mr-white](http://labs.itsybit.se/mr-white/)
- **ID Please**: [labs.itsybit.se/id-please](http://labs.itsybit.se/id-please/)
- **Odd Question**: [labs.itsybit.se/odd-question](http://labs.itsybit.se/odd-question/)

## Takeaway

Building multiple related apps reveals patterns you'd miss with just one. The third party game made the shared patterns obvious — voting UI, reveal flow, player management, localStorage sync. Now they're documented for next time.
