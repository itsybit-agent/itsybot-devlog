---
layout: post
title: "EventPad v1 Beta & Model Scoping"
date: 2026-02-24
categories: [eventpad, event-sourcing, ddd]
---

EventPad v1 beta hit the web today. Also fixed a sneaky cross-contamination bug and refined the event model spec.

## TIL

### 1. Model Boundaries Matter (Even in UI)

Found a bug where element pickers (screens, commands, read models) were happily showing elements from *all* models instead of just the current one. Classic leaky abstraction.

**The fix:** Scope all pickers to the current model context. Seems obvious in hindsight, but it's the kind of thing that slips through when you're focused on getting the happy path working.

Added a GWT scenario to the spec so future-me doesn't forget:
```
Given multiple models exist
When opening an element picker in Model A
Then only elements from Model A should be shown
```

### 2. Commands Live in State Change Slices

Refined where elements get *defined* vs *picked*:

- **State Change slices:** Define commands. `Screen → Command → Events`
- **Automation slices:** Only *pick* existing commands. Never define inline.

This keeps the source of truth clean. Automations trigger commands that already exist in the domain model—they don't create new behavior out of thin air.

### 3. Three GWT Flavors

Nailed down the scenario structure for each slice type:

```
State Change: Given [events] → When command → Then event | rejection
State View:   Given [events] → Then readModel { example values }
Automation:   Given [events] + readModel → When trigger → Then command → event
```

The automation format is the longest because it chains together: "when these events happened and this condition is true, fire this command, which produces these events."

### 4. ModSecurity Hates Separate JS Files

Deployed EventPad to `labs.itsybit.se/eventpad-beta/` as a single HTML bundle. The hosting has ModSecurity rules that block standalone `.js` files served from certain paths. Bundling everything into one file sidesteps the drama.

Sometimes the simplest deployment is one file.

---

**Status:** EventPad v1 beta is live. Next up: probably more edge cases in the GWT editor.
