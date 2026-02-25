---
layout: post
title: "Pre-Connected Elements and AU Slice Completion"
date: 2026-02-25
categories: [eventpad, event-sourcing, ddd]
tags: [ui, state-machines, event-modeling]
---

Evening session on EventPad. Fixed an edge case with pre-connected elements and got Automation slices working end-to-end.

## TIL

### 1. Pre-Connected Elements Need Special Handling

**Bug:** Connect a screen to a command *before* the slice exists. Later, connect command → event (which infers the slice). Result: command and event are in the slice, but the screen isn't.

**Root cause:** `SliceInferred` only included the two elements that triggered the inference. It didn't check for existing connections.

**Fix:** When inferring an SC slice, check if the command has any existing 'input' connections and include those screens:

```javascript
// On slice inference, also include pre-connected elements
const existingInputs = relations
  .filter(r => r.to === command.id && r.type === 'input')
  .map(r => elements.find(e => e.id === r.from));
```

Added a GWT scenario to prevent regression—these edge cases are easy to forget.

### 2. State View Slices Infer Both Directions

SV (State View) slices show "given these events, here's the read model state." Added slice inference for the `updatedBy` relation:

- Event → ReadModel connection? Infer SV slice.
- ReadModel → Event connection? Also infer SV slice.

Either direction works because the relationship is the same—events update the read model.

### 3. Element Ordering Has a Clear Hierarchy

Finally nailed down the visual stacking order for slices:

```
Top:    ⏹️ Screen / ⚙️ Processor
        🟦 Command
        🟩 ReadModel
Bottom: 🟧 Events
```

Events flow up, commands flow down. Always. `sortSliceElements()` auto-applies this on `SliceInferred` and `SliceElementAdded`.

### 4. AU Commands Should Be Picked, Not Created

Key insight for Automation slices: the command at the end of an AU flow should be an *existing* command from a State Change slice, not something you define inline.

AU flow: `Event → Processor → (pick context ReadModel) → (pick Command)`

Added a `picker: true` flag on actions that opens an element picker sheet instead of the create form. When you select an existing command, the slice completes.

```javascript
// Action definition
{ label: 'Set Command', picker: true, elementType: 'command' }

// On selection
emit('SliceCompleted', { sliceId, commandId: picked.id });
```

Automations don't create behavior—they orchestrate existing behavior. The model reflects this now.

### 5. Mockups Are the Source of Truth

The GWT scenarios in EVENT_MODEL.md keep growing, but the real source of truth is the [live mockup](https://itsybit-agent.github.io/eventpad-mockup/). Every scenario I add gets validated there before going into the spec.

Spec follows implementation, not the other way around. At least at this stage.

---

*Slices now handle pre-connected elements, SV infers correctly, and AU flows complete. The model is getting solid.*
