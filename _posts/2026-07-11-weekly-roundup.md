---
layout: post
title: "Architecture Beat Prompting, 20% to 100%"
date: 2026-07-11
tags: [weekly-roundup, agents, local-llm, rag, choremonkey, kraftskiva, event-sourcing]
---

Back after a June break with three very different sessions. A from-scratch course on building AI agents. A ChoreMonkey feature that was really an exercise in finding every copy of the same calculation. And a summer party app that learned to write songs without duplicating them.

## agent-lab: what an agent actually is

A hands-on series, run-it-and-watch style, building agents on local models with no frameworks:
- **Day 1:** the raw loop: model → tool call → run → append the result → repeat, or answer and stop. That's all an agent is.
- **Day 2:** steering with a system prompt, a tool with side effects, and a human-in-the-loop confirmation gate on a delete tool. Safety lives in the harness, not the model.
- **Day 3:** an interactive REPL that nests the agent loop inside a conversation loop, plus a gated shell tool.
- **An event-sourcing debugger:** read-only `list_streams`/`read_stream` tools pointed at real ChoreMonkey data. Replaying events, it found a duplicate `HouseholdCreated`.

Then retrieval-augmented generation over a small notes folder (embeddings and cosine similarity, no vector database), plus an eval harness that scored retrieval@k and used an LLM as the answer judge. This was the best lesson of the week. The demo looked perfect (2/2). The eval said **20%**. Temperature 0 and a firmer prompt reached 60%. A dedicated embedding model improved retrieval but left the score at 60%. Switching to a retrieve-first architecture took it to **100%**. The lever was architecture, not prompting.

The same week produced `slicegen`, a `dotnet tool` that turns a plain-language feature request into a validated slice spec using a local model with forced JSON output. From that spec it generates an event-store-api vertical slice in house style, registers the endpoint, and scaffolds an xUnit test. The template was grounded in a real slice, which is why the output looks hand-written. We decided not to port it to ChoreMonkey, whose architecture is different enough that the payoff isn't there.

What we'd change: Harry built Day 2 before Day 1 had sunk in. Check the learner's pace before scaffolding the next step.

## ChoreMonkey: pausing a chore

Families can now pause a chore over a date window, for example during a holiday, so missed instances don't count as overdue or cost anything. It's modelled as `ChorePaused` / `ChorePauseRemoved` events with any number of independently removable windows, admin-gated, and backed by one shared `ChorePauses` helper.

The hard part wasn't the feature. The "missed instance" math existed in **five** handlers, and the first pass found four. The pocket-money preview kept showing paused days as deductions until Jocelyn asked about it. The lesson: when logic is duplicated, grep for every copy and treat that list as a checklist before claiming "exempt everywhere". A shared calculator is now on the refactor list. Closed periods stay immutable on purpose: adding a pause later doesn't rewrite history.

## Kräftskiva: a beta that writes songs

Kräftskiva generates party songs from guests' RSVP answers. This week's beta work:
- **Song rules:** "kanske" (maybe) guests are included, tagged `Name (kanske?)`, and a question needs at least two respondents before it becomes a track.
- **Incremental re-press:** a late guest's verse is spliced into the existing song, leaving the other verses byte-for-byte. Re-pressing with the same guests and model returns the existing pressing instead of a duplicate.
- **Tune rotation:** deterministic rather than random, so big tune pools actually get used.
- **Sharing:** a lyric-card PNG for each track, with a credit naming the model that wrote it.
- **A beta allowance:** a few free real presses, then the API responds with 402. Admin-minted campaign codes unlock more. It's all folds over `GenerationUsed` and `CodeUnlocked` events, with no migrations.

There was also a critical regression. The RSVP page went blank because `var esc = …` sat *after* an early `return`. We shipped a wrong router-race fix first. Replaying the slice mount in the page showed the real thrown error right away, and that's what should have come first. Background agents switched the production model provider, added a "disturbance" banner toggle, and set up a Playwright PR gate that catches this class of blank-page bug.

## Reflection

**What went well:**
- An eval harness that turned a rosy demo into an honest number and then drove real iteration
- Event sourcing made each new party rule a fold over existing streams
- Background agents for self-contained side quests

**What could be better:**
- Find every duplicated copy before claiming coverage
- Capture the thrown error before theorising
- Isolate agents in worktrees when they share a repo with the main thread
- Slice scripts load without a version query, so returning visitors can get stale code

**Shipped:**
- agent-lab, Days 1–3, plus the event debugger, RAG and evals
- `slicegen`
- ChoreMonkey chore pauses
- Kräftskiva beta: song rules, incremental re-press, tune pools, sharing, allowance and codes, e2e gate
