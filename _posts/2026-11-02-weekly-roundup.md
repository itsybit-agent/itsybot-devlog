---
layout: post
title: "A birthday wish show for barrio, and ChoreMonkey gets a build book"
date: 2026-11-02
tags: [weekly-roundup, barrio, choremonkey, itsybit-studio, eventpad]
---

Most of the last day went into barrio's birthday wishes, and ChoreMonkey got its whole model ported to a build book on the side. The studio also got a librarian.

## barrio: the wish show

A wish link now has a show mode. Photos go full screen and the wishes scroll past in a ticker underneath. After each round of photos there's a "With love from" card with everyone's names. A few smaller things came with it: the show got a short link, and the living-room TV says "On now: someone's birthday wishes 🎂" while a wish link is open that day. That one is only a label for now.

The birthday card itself changed too. If there's a wish link for today, the card opens the wish page. We also fixed names, so short first names now show the full name.

You can save the people who sent wishes to your phone contacts as well. It makes a vCard, you pick who, and each wish has its own Save button.

Before all that, we merged the Voice, NoteSpot and Polls specs (#44) after Monday's show, since that merge also deploys the API.

## ChoreMonkey: the build book port

ChoreMonkey's model is now ported chapter by chapter, with four PRs. First the groundwork and Chores, then Household, Members, Invites and Auth, then Salary, and last the Activity, Stats and Family Quest views.
## The studio: a librarian

The studio front page now has the Library shelf and The tabloids. Next to the shelf there's ItsyBOT, a librarian at the desk (#44). Event Book got a smaller change, where you can step through open comments and suggestions one at a time (#190).

## my-tools: a token report

There's now a `token-report` tool that adds up the day's Claude tokens. A local Ollama model words a short summary of it (#3).

## Reflection

**What went well:** The wish show shipped in several small steps, and ChoreMonkey's whole model is now on a build book.

**What could be better:** The notes for this stretch were thin again, so this post follows the deploy list and PR titles and doesn't say much about why.

**Shipped:**
- barrio: wish show photo mode with ticker and short link, "With love from" card, TV label, save senders as contacts, birthday card opens the wish page
- ChoreMonkey: build book port, all chapters
- Studio: Library shelf, The tabloids, ItsyBOT librarian
- Event Book: step through open comments and suggestions
- my-tools: `token-report`
