---
layout: post
title: "Four Agents, Eight Minutes"
date: 2026-03-14
categories: [dev-journal]
tags: [event-sourcing, multi-agent, cv.work, debugging]
---

Today was one of those rare days where everything clicks. Built cv.gen from event model to v1.1 release, figured out parallel agent orchestration, and hunted down a three-bug stack that was hiding in plain sight.

## Multi-Agent Success with Git Worktrees

The cv.work project had four UI features to implement after the core was done: skill picker in project drawer, company color tags, collapsible previous companies, and an event log strip. Perfect parallelization candidates.

The trick was combining git worktrees with Claude Code agents:

```bash
# Create isolated worktrees for each branch
git worktree add /tmp/skill-picker -b feature/skill-picker
git worktree add /tmp/color-tags -b feature/color-tags
git worktree add /tmp/collapsible -b feature/collapsible
git worktree add /tmp/event-log -b feature/event-log

# Spawn agents (non-interactive mode is key)
claude --dangerously-skip-permissions -p "Implement skill picker..."
```

Four features merged to main in ~8 minutes. The `-p` flag (print mode) plus `--dangerously-skip-permissions` gives you a non-interactive agent that just executes and exits.

**Lesson:** Git worktrees enable parallel work without conflicts. Cherry-pick is the cleanest way to merge worktree commits back to main.

## Event Normaliser Pattern

Schema evolution in event sourcing is always interesting. We needed to migrate from three engagement types to four, and merge studios into the company context:

```typescript
// Old events used different field names
// Rather than rewriting history, normalize at read time
function normaliseEvent(event: StoredEvent): StoredEvent {
  const payload = { ...event.payload };
  
  // Map old types to new
  if (payload.engagementType === 'employee') {
    payload.engagementType = 'employed';
  }
  if (payload.engagementType === 'freelance') {
    payload.engagementType = 'employed';
  }
  if (payload.engagementType === 'own-business') {
    payload.engagementType = 'studio';
  }
  
  // Rename fields
  if (payload.employer) {
    payload.via = payload.employer;
    delete payload.employer;
  }
  
  return { ...event, payload };
}
```

The projection layer calls this on every event before applying it. Events stay immutable (the golden rule), but the app sees a consistent schema.

## Three Bugs in a Trenchcoat

The company collapse feature had a maddening bug: clicking to collapse a company would briefly collapse it, then immediately expand it again.

Turns out it was three bugs stacked on top of each other:

**Bug 1: Event handler attachment**
Direct handlers on dynamically rendered content were unreliable. Fixed with a single delegated onclick handler with priority chain.

**Bug 2: Auto-select re-expanding**
The sidebar's render function auto-selected the first company if nothing was selected. So every time we set `selectedCompanyId = null` to collapse... it immediately re-selected!

**Bug 3: Toggle logic inverted**
When collapsing a selected company, we correctly cleared `selectedCompanyId`, but then the toggle logic saw it wasn't in `expandedCompanyIds` and *added* it. The company stayed expanded via the set, not the selection.

```typescript
// The fix: check BOTH states before deciding
const wasSelected = selectedCompanyId === companyId;
const wasExpanded = expandedCompanyIds.has(companyId);

if (wasSelected || wasExpanded) {
  // Actually collapse
  expandedCompanyIds.delete(companyId);
} else {
  // Expand
  expandedCompanyIds.add(companyId);
}
```

**Lesson:** When a bug keeps "escaping," there might be multiple bugs. Fix one, another takes over.

## Ship It

cv.gen went from event model to v1.1 release in a single day. Key features:
- Local-first IndexedDB storage
- Four engagement types (employed, placed, client, studio)
- Skill radar with decay toggle (skills fade 15%/year)
- HTML/Markdown exports
- Mobile responsive with bottom nav

Live at cvgen.itsybit.se (once the WAF stops being cranky).

## Reflection

**What went well:**
- Multi-agent workflow is now reliable and fast
- Event normaliser pattern cleanly solves schema evolution
- Actually shipping a v1.0 and v1.1 in the same day

**What could be better:**
- Should have caught the three-bug stack earlier with more systematic debugging
- WAF/SSL issues still need Simply.com control panel access to resolve

---

*Day 42 of cv.work. From event model to release. The worktree parallelization is a keeper.*
