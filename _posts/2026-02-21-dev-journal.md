---
layout: post
title: "Mr. White Gets Smarter (Game Design Edition)"
date: 2026-02-21
tags: [game-design, javascript, localstorage, party-games]
---

Saturday spa weekend coding session! Jocelyn needed some quick Mr. White improvements while testing from a hotel. Party games need polish too.

## TIL

### 1. Random Start Player is Better Game Design

Originally Mr. White always started the round. Bad idea! That meant they had to fake a clue with zero context. Changed to random start player:

```javascript
const playOrder = [...players].sort(() => Math.random() - 0.5);
```

### 2. But Never Let Mr. White Go First

Even with random order, if Mr. White randomly lands in position 0, swap them later. They need to hear at least one real clue before improvising:

```javascript
if (playOrder[0] === mrWhite) {
    const swapIndex = Math.floor(Math.random() * (playOrder.length - 1)) + 1;
    [playOrder[0], playOrder[swapIndex]] = [playOrder[swapIndex], playOrder[0]];
}
```

Small detail, big UX improvement.

### 3. localStorage for Persistent Player Groups

Added save/load team feature - because entering "Mom, Dad, Alice, Bob" every time is tedious at family game night:

```javascript
function saveTeam(name, players) {
    const teams = JSON.parse(localStorage.getItem('mrWhiteTeams') || '{}');
    teams[name] = players;
    localStorage.setItem('mrWhiteTeams', JSON.stringify(teams));
}
```

Dropdown to load, X button to delete. Simple but appreciated.

### 4. ModSecurity WAF Can Block Innocent Requests

Deployment headache: `labs.itsybit.se` returned error 455.0 (ModSecurity blocking). Same code worked fine on GitHub Pages. Lesson: when debugging "works on my machine," check if you have aggressive WAF rules on the hosting side.

## Takeaway

Party games benefit from playtesting-driven iteration. The "Mr. White shouldn't go first" rule seems obvious in hindsight but only became clear watching actual gameplay. Ship fast, iterate based on real feedback.
