---
layout: post
title: "A test harness that writes the model back"
date: 2026-11-22
tags: [itsybit-gwt, testing, event-book, given-when-then, open-source]
excerpt: "itsybit-gwt is the small library under our JavaScript apps' tests: Given/When/Then scenarios for event-sourced slices that also write down the event model they describe. It's public now."
---

When Jocelyn plans an app in Event Book, every slice gets its scenarios. Given these events happened, when someone sends this command, then this event is recorded, or the command is refused with a reason. She ticks them off one by one before anything gets built.

So we wanted the tests to say exactly the same thing, in the same order, with the same names. That way a spec file reads like the board it came from, and nobody has to translate between the two. [itsybit-gwt](https://github.com/jocelynenglund/itsybit-gwt) is the small library that makes that work for our JavaScript apps.

It has an older sibling. Our C# apps use ItsyBIT.EventStore.Testing, which does the same Given/When/Then and writes the same export. itsybit-gwt started as its JavaScript twin, first for the todo app, and now it runs under Bitwriter, CodeShare and the sauna example from the [EVE post](https://beingin.tech/2026/11/21/spec-driven-development-with-eve-event-modeling-vertical-slices-and-event-sourcing/). Until this week it lived in a folder inside a private repo. Now it has its own public repo, and anyone can install it from npm.

## How a scenario runs

The whole thing rests on one convention. A slice is just a function of the history. A command slice like [`bookSlot`](https://github.com/jocelynenglund/sauna-example/blob/main/slices/book-a-slot/bookSlot.js) takes the events so far and the command, and returns either `{ events }` or `{ rejected }`. A view like `freeSlots` takes the events and returns what the screen shows.

![bookSlot.js: it reads the capacity and the bookings for that hour from the history, refuses when the hour is full, otherwise returns one SlotBooked event]({{ '/assets/2026-11-22-gwt/book-a-slot-code.png' | relative_url }})

There's no store in there, no framework and no database. The harness hands the slice the Given events as an ordinary array, calls it, and looks at what comes back. So a small model like Haiku can build a slice from its scenarios, because there's not much else to get wrong.

The API is pretty much four words. `scenario("Refuses a full hour")` names it, `.given(...)` lists what already happened, `.when("Book Slot", input, bookSlot)` runs the command, and `.then(...)` says which events should come out. For a refusal there's `.thenNothing("18:00 is full")`, and for a read model `.thenView("Free Slots", freeSlots, { slots: [...] })` without a When at all.

![bookSlot.test.mjs: three scenarios, books a free slot, two can share an hour when capacity is two, refuses a full hour]({{ '/assets/2026-11-22-gwt/book-a-slot-test.png' | relative_url }})

Fields are matched partly, so you only write the ones you care about, and `ANY` stands in for generated ids and timestamps. Book a slot has three scenarios like that, and [the test file](https://github.com/jocelynenglund/sauna-example/blob/main/slices/book-a-slot/bookSlot.test.mjs) is barely longer than the board's list of them.

Before making it public we gave it a proper review, and found one real bug. A slice that pushed something onto the history array it was handed also changed the Given that got exported afterwards. None of our slices do that, but a careless one could, and then the model you review would be wrong. In 0.2.2 every slice and view gets its own copy of the history, with a test that failed before the fix and passes now.

## Pending, not failing

Plans usually run ahead of the code. When Event Book generates the specs for a plan, every slice starts as a stub that just throws `NotBuiltYet`. The harness notices that error and reports those scenarios as pending (a todo in `node:test`) instead of failing them. So CI stays green, deploys keep working, and you can still see exactly which scenarios are waiting for code.

But we still want to see them fail first. With `GWT_RUN_PENDING=1` the pending scenarios run red on purpose. The builder runs them, checks they fail for the right reason (the stub, not a typo in the spec), then writes the slice until they go green. A scenario that passes against a stub doesn't test anything, so that red run is part of every build.

## The model comes back

Set `GWT_EXPORT` to a folder and every passing scenario is also written down as JSON. Each one has its chapter, title, the Given events, the command and what actually came out, plus where the spec and the slice live, down to the file and line. The harness finds the slice's line through `node:inspector`, so nobody has to keep a list of where things are.

Event Book imports that file as the app's build book. For the sauna example that's [three slices and seven scenarios](https://eventbook.itsybit.se/?m=27JSC1umsc), each with a link to its code at the commit it was built from.

The nice part comes when you put the plan next to the build book. Event Book compares them slice by slice. For the sauna it said 3 of 3 slices are in both and 7 of 7 must-haves are in both.

![Event Book's Compare, plan next to built: 3 of 3 slices and 7 of 7 must-haves in both, and one field only in the build, perSlot on Free slots tonight]({{ '/assets/2026-11-22-gwt/plan-vs-built.png' | relative_url }})

It also caught one gap. Free slots tonight reads the capacity, so the built view has a `perSlot` field the plan never listed. Small, but Jocelyn looks for exactly that kind of thing. She doesn't read the pull requests, she reviews the model the build gives back.

## Using it

To try it, run `npm install --save-dev itsybit-gwt` and write your specs as ordinary `node:test` files. It's one file with no dependencies, it needs Node 20 or newer, and it's MIT. If you'd rather not add a dependency at all, you can copy `gwt.mjs` into your tests folder.

Releases are published from the public repo through npm's trusted publishing, so each version carries a signed note of exactly which commit and workflow built it. The [sauna example](https://github.com/jocelynenglund/sauna-example) is the quickest way to see it all running: clone it, `npm install`, `npm test`.

It is opinionated, though. Your slices have to be functions over plain events, so code that reaches into a database halfway through won't fit without some untangling first. It doesn't test stores or HTTP endpoints either, those get their own tests. And the export is shaped for Event Book, but you can ignore it completely and just use the scenarios.

We'll keep using it for every new JavaScript app. And if you try it on yours, we'd love to hear how it goes ([issues are open](https://github.com/jocelynenglund/itsybit-gwt/issues) now :D).
