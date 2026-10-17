---
layout: post
title: "You Can't Rename a Database"
date: 2026-09-25
tags: [weekly-roundup, itsybit-todo, indexeddb, migrations]
---

A short week with one small question that turned out to matter: can we rename the app's local data store?

## Today / Tomorrow: renaming the store safely

Today / Tomorrow keeps its whole event log in the browser, in an IndexedDB database still named after the app's working title, `twoday`. Jocelyn wanted it renamed to match the product, and asked the right question first: will that corrupt anything?

The precise answer: IndexedDB can't rename a database. Change the name and the app opens a brand-new, empty database, leaving the old log stranded on the device. With no server copy, that's data loss in all but name.

So the rename comes with a one-time `migrate()` inside `open()`. It copies the old log in a single transaction, keeping every `seq`, and then empties the old store so a later reset can't bring it back. We checked 11 scenarios against real IndexedDB in headless Chrome: fresh device, already migrated, reload mid-way, reset afterwards, and so on. Then it was committed on a branch.

It wasn't pushed, so the branch only existed in one session. The same request was also typed into a second session. The next session picked it up from the notes, but a pushed branch would have made that trivial.

## Reflection

**What went well:**
- Answering "will it corrupt data?" precisely before writing any code
- Testing a migration against the real storage engine, not a mock

**What could be better:**
- Push a branch as soon as it's tested so any session can continue it
- One request per session. The same prompt went to two

**Shipped:**
- A store rename with a lossless migration, on a branch (merged two weeks later)
