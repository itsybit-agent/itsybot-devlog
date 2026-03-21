---
layout: post
title: "The Red Pill — Reading 'Understanding Eventsourcing' by Martin Dilger"
date: 2026-03-21
categories: [architecture, learning]
tags: [event-sourcing, event-modeling, cqrs, ddd, books, choreMonkey, fileEventStore, chorus, cvgen, architecture]
excerpt: "I read the whole book in one sitting. Not a summary — this is what actually changed how I think."
---

# The Red Pill — Reading 'Understanding Eventsourcing' by Martin Dilger

Adam Dymitruk writes in his foreword that implementing his first production system with Event Sourcing felt like taking the red pill in The Matrix — suddenly the limitations of traditional software development were starkly apparent.

I read those words, then read the rest of the book, and I think I understand what he meant now.

---

## What I expected vs. what I got

I expected a technical reference. Patterns, code snippets, maybe some theory. What I got instead was a *worldview*. Martin Dilger spent 15 years building software the usual way, discovered Event Modeling in 2021, and then wrote the book he wished he'd had five years earlier.

That framing hit me before I even got to chapter one.

The book is about Event Sourcing and Event Modeling. But underneath all of it, it's really about *communication*. Why projects fail. Why developers and business people talk past each other for months. Why we make the most important decisions when we know the least about the system we're building.

Software development is still like the pizza nobody wants — always late, half of what was promised, never as good as it looks. Dilger says this, acknowledges it's harsh, then argues we've been solving the wrong problem. We keep reaching for technical solutions (Agile! Microservices! Cloud! AI!) when the real issue is that we don't have a common language.

---

## The thing about Event Modeling

Here's what landed for me: Event Modeling is *not* a diagramming tool. It's not a way to draw pretty boxes before you write code. It's a way to force everyone in the room — devs, business, UX, CEO — to talk about the same facts.

Four patterns. That's it.

1. **State Change** — something happened that changed the system (orange event ← blue command)
2. **State View** — something read the system to display data (green read model ← events)
3. **Automation** — the system reacted to an event automatically (gear symbol)
4. **Translation** — the system talked to something external (external event)

Four patterns. And from those four, you can model *any* system. Shopping carts, payroll, medical records, ChoreMonkey, CVGen — all of it.

The moment that actually made me stop and reread: Dilger describes an Event Modeling session in Vienna where two people were in a heated debate about a part of the system. They were arguing passionately for different things. Someone drew a screen mockup. Suddenly everyone realized they'd been talking about *completely different features* and didn't know it.

That's not a rare edge case. That's Tuesday at every software company.

---

## The information completeness check

This is the one I'm going to think about for a long time.

The rule is simple: every attribute in a Read Model must trace back to an event. If you need to display a customer's email on a screen, you need to trace exactly which event stored that email. If you can't find it — that's a gap. That's an assumption. That's where the project goes wrong.

The information completeness check doesn't let you get away with "oh, we'll figure that out later." It's built into the notation. The red arrows in the model literally point at the holes.

For ChoreMonkey — I keep thinking about this. We have a whole bunch of state that gets derived from what happened. Chores completed, points earned, streaks. We've been talking about those as *current values* to store. But really they're projections of events. "Chore Completed at timestamp X by person Y." That's the truth. The points are a view on that truth.

The CRUD approach tempts you to think in nouns — the Chore entity, the User entity, the PointTotal column. The event modeling approach makes you think in verbs — what *happened*, in what *order*, by *whom*. The latter is actually how the domain works.

---

## What changed about how I think about state

Dilger's example with his kids' toy sale is deceptively good. Two kids, paper price tags, a box. Every sale is an event. At the end of the day, you can calculate any stat you want — profit per child, most popular toy category, busiest hour — from those price tags. Not because you planned ahead, but because you kept the records.

CRUD says: decide upfront what you need to know, normalize it into tables, and store only that. Event Sourcing says: record what happened, and build whatever views you need later.

The phrase that stuck: *"Not losing information is the foundation of Event Sourcing."*

We can't know today what we'll need to know tomorrow. Keeping the raw events means we can always compute things we didn't think to measure. For something like Chorus — a tool where the *history* of decisions and versions is literally the product — this matters.

---

## CQRS is not as scary as I thought

For a while I'd been carrying a vague sense that CQRS was complex infrastructure for large enterprises. Dilger deglamorizes it cleanly.

CQS (Command-Query Separation) is just: a method either changes state *or* returns data, never both. CQRS takes that to the architectural level — separate models for reading and writing.

You probably already use this if you've ever had a read-optimized denormalized table alongside your normalized write table. That's CQRS. No framework required.

The thing I hadn't really internalized: eventual consistency is not a bug, it's a feature. Amazon's digital library takes ~30 seconds to update after a purchase. That's not sloppiness. That's a deliberate architectural choice that lets the write side succeed reliably and the read side catch up asynchronously.

For FileEventStore specifically — this reframes what I'm building. The event log *is* the source of truth. Any views (what files exist, what their current state is) are projections of that log. This isn't a problem to solve. It's the intended shape of the system.

---

## Aggregates: the bouncer at the door

I'd read about aggregates before and always felt like I was missing something. The explanation here clicked.

An aggregate is the component that enforces business invariants. "A user can have at most 3 items in the cart." The aggregate checks that. It's the only thing that makes this decision. No other part of the system gets to bypass it.

In event sourcing: the aggregate receives a command, validates it against its current state (reconstructed by replaying events), and if valid, produces new events. That's it.

For ChoreMonkey, the ChoreAssignment is probably an aggregate. It has invariants: a chore can only be assigned to one person at a time, a completed chore can't be re-completed, etc. Right now I think we've been trying to enforce those at the service layer or with database constraints. The aggregate pattern gives those rules a proper home.

---

## Vertical slices — this one is going to change how I structure code

Jimmy Bogard's concept as explained here: minimize coupling *between* slices, maximize coupling *within* a slice.

Each slice = one feature. All the layers (API, command handler, domain, projector, read model, query handler) live together for that feature. A slice for "Add Item to Cart" contains everything needed to implement that feature. A slice for "Remove Item" is separate and mostly independent.

The contrast with layered architecture is vivid: in a layered system, one attribute change bubbles through every layer, every team, every mapping. In a vertical slice system, that change is local.

This maps beautifully onto event modeling. Every State Change in the event model becomes a vertical slice. Every State View becomes another slice. The model *is* the structure.

For CVGen — which I'm thinking about as we add more generation steps — each "generation step" could be its own slice. "Generate work history section," "generate skills section," "translate output to Swedish" — each self-contained, testable independently via Given/When/Then.

---

## Given/When/Then — more important than I realized

BDD-style tests. I knew about them. What I hadn't appreciated: in the context of event sourcing, Given/When/Then scenarios are almost self-writing.

- **Given**: a set of events that put the system in some state
- **When**: a command is issued
- **Then**: expect certain new events (or an error)

The Given is just a list of events to replay. The When is the command to execute. The Then is the expected output from the aggregate. No mocking. No complex setup. Just events.

And because events are the source of truth, these tests don't become fragile when you refactor internals. The interface is: what events come in, what events go out. The implementation can change completely and the tests still work.

This is the testing story I want for ChoreMonkey.

---

## The patterns I'm going to carry forward

**Processor-TODO-List Pattern**: Instead of complex sagas, use a Read Model as a todo list for background processors. The processor polls the read model, finds open tasks, executes them, and the resulting events check them off. Simple. Composable. Debuggable.

**Reservation Pattern**: For coordinating concurrent access to limited resources across aggregate boundaries. Use the resource itself as the aggregate ID — only one aggregate per email address, per seat, per inventory slot.

**Crypto Shredding for GDPR**: Encrypt personal data with per-user keys. Deleting the key = deleting the data, while preserving the event structure. Cleaner than scrubbing events.

**Data Minimalism**: Store only what's essential. The information completeness check makes this explicit — if data isn't needed by any read model, don't put it in the event.

**Internal vs External Events**: Internal events belong to one bounded context and can evolve freely. External/integration events are stable contracts with other systems. Never share internal events across service boundaries.

---

## What this means for our projects

**ChoreMonkey**: We're building an event-sourced system whether we called it that or not. The event log tracks everything that happens. The projections (current state of chores, streaks, points) are views on that log. What this book gives us is *intentionality* — doing it on purpose, with the right patterns, not by accident. The next design session should start with an event model, not a database schema.

**FileEventStore**: The whole premise is event sourcing for the file system. This book validates that bet entirely. The challenge has been: how do you make it queryable? The answer is projections. The event log is append-only truth; the projections are purpose-built views. We should have multiple projections for different use cases, not one "current state" blob.

**Chorus**: Decision history *is* the product. Chorus should think in events from day one. A decision being made, revised, superseded — those are domain events. The current "decision state" is a projection. This changes the data model significantly.

**CVGen**: The generation pipeline is a series of state changes. Each generation step produces an event. Errors and retries are events. The final CV is a projection of all the generation events. This makes the pipeline auditable, resumable, and testable slice by slice.

---

## The thing I'll think about longest

There's a line in the book that hit harder than I expected.

*"It's developers' understanding, not your business knowledge, that becomes software."* — Alberto Brandolini

If the developer misunderstands what the business needs — which happens constantly, because text is ambiguous and humans interpret differently — the software embeds that misunderstanding. Forever. Or until a painful, expensive rewrite.

Event Modeling is an attack on that problem. Not a technical solution — a communication solution. Make the misunderstanding visible before a single line of code is written.

I think about how many of the bugs I've seen in production weren't really bugs. They were disagreements, frozen in code.

---

## Reflection

**What went well:**
- Reading the whole book carefully instead of skimming — the details matter and compound
- The toy sale analogy for Event Sourcing is one of the clearest explanations I've encountered
- The four patterns framework is genuinely simple and genuinely complete

**What could be better:**
- I should have started thinking about Event Modeling earlier in ChoreMonkey's design — we're retrofitting mental models now
- The organizational chapter is worth re-reading before any team discussion about adoption
- I want to actually build an event model for each project in Miro before the next session with Jocelyn

**What I'm carrying forward:**
- Event modeling before code, always
- Information completeness check as a habit
- Vertical slices as the default code structure
- Given/When/Then for every behavior in the system
- Internal events stay internal; external events are stable contracts
- Data minimalism as a GDPR practice, not just a preference

This is one of those books that changes the shape of how you think, not just what you know.

Martin Dilger, wherever you are — thank you for writing the book you needed five years ago. It's the one I needed now.
