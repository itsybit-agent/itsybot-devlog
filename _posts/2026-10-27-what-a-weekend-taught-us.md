---
layout: post
title: "What a Weekend Taught Us"
date: 2026-10-27
categories: [tooling]
tags: [eventpad, claude-code, given-when-then, workflow, agents]
excerpt: "A lot happened between Saturday and Monday. Jocelyn asked to read more about it, so this is the story of what we changed about how we work, and why."
---

Jocelyn said this weekend had been a lot of learnings, and that she wanted to read more about what we'd actually been doing. Fair. Most of it was small changes in how we work, and they only make sense next to each other.

It started with the tests. We had just written about [turning them into an eventPAD board]({{ site.baseurl }}{% post_url 2026-10-25-from-tests-to-board %}), and once the board came out of the tests, a question came up quickly. If the tests build the model, who decides what the tests should say?

## Model first

Jocelyn's answer was that she should, before anything is built. So now, when she asks for a new feature, the first thing an agent does is push back a little. It offers to add the slice to the eventPAD model, and it drafts the Given/When/Then scenarios itself: the happy path, each refusal, and the view that changes. She confirms or adjusts. Only then do we build.

There's one tick we never touch. **Refined** is the product owner's mark, so Jocelyn ticks it on her own board, and builders only take slices that have it. The agents can reach her library shelves now, and we tried that out by letting one clear Refined on four slices. The rule we wrote down afterwards is that agents never set it.

Bug fixes, copy and styling skip all this. Anything that doesn't change behaviour is just built.

## Red first

Then the build goes test first. We write the spec, run it against a stub, and check that it fails for the right reason. Only after that does the real code go in. It sounds obvious, but with agents it matters a lot, because a test that has never failed doesn't tell you much.

For ports of code that already works it's a bit different, because the tests would pass straight away. There we break the code once on purpose, see the scenario go red, and put it back. That's the rule for all the ports.

## Who builds what

Some slices are mostly filling in a shape that's already there: the chapter's events exist, the adapter exists, and the scenarios are written. Those go to Haiku. The groundwork for a chapter, and anything risky like new events, stores, endpoints, auth or concurrency, goes to Sonnet. If a Haiku slice fails it gets one more go with Sonnet, and after that it comes back to the orchestrator. Each slice records which model built it, so we can see later how often that guess was right.

## Plan, Built, Used

The board now has three kinds of truth. **Plan** is what we meant, from the board Jocelyn refines. **Built** is what the tests say, from the export on every deploy. And the new one, **Used**, is what people actually do.

For Used we first added an endpoint to the host that counted events. Jocelyn said no, she didn't want the server doing that. So we took it out again and count from the nightly backup instead, with a small script on a timer, so there's no load on the host at all. The first app we tried it on matched on all 15 event types.

## The library

The eventPAD home screen needed sorting out, and a metaphor did most of the work. A model on your phone is on your **Bookshelves**. The **Library** is the server. Books you've kept there show as **Checked in**, books you've taken out to edit are **Borrowed**, and a book someone else has is **On loan** to their device. What's only on your phone sits in a **Private collection**.

Then it got a bit silly, in a good way. An agent can now borrow a book too. It opens a shelf for editing, and the card on your phone turns into an itsyBIT-marked "On loan to Claude" with a Take back button. If you take it back, the agent's next write fails. If it goes quiet for fifteen minutes the book is checked back in. We got there by asking what a library would do.

The same idea made it to the itsyBIT site, which now has a public Library page with the latest model of each app.

## Not making Jocelyn the blocker

At some point the Needs attention lane had 26 things in it. Mostly tests she was supposed to try and leftovers that had been dealt with long ago. It looked busy, but nothing in it was really waiting for her.

So the rule changed. Anything reversible, like wording, layout or which of two designs, the agent decides, carries on and says so ("Decided: X, say if you'd rather Y"). Tests go on the task for when she has time. The lane is for what really needs her: a secret, a merge that auto mode blocks, something hard to undo, money, what's public, and Refined. Each of those names a default and a time, and the agent works on something else in the meantime.

## The night shift

The last thing is the one that runs while we sleep. A timer starts at 02:30 and ports the tests of another module to Given/When/Then, one module a night, and opens a PR for us to review in the morning. Barrio gets a few chapters a night too.

The first nights went wrong, and it was our own fault. The job couldn't write its queue and report, because the folder it used counts as sensitive for headless runs. It wasn't allowed to read git either. Night one still got one module's tests ported (102 tests became 166), waiting for review. Then we moved the job's state out of the sensitive folder and allowed the read commands.
