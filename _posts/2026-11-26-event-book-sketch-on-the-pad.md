---
layout: post
title: "Event Book: sketch on the Pad first, then make the model"
date: 2026-11-26
tags: [event-book, pad, event-modeling, agents]
excerpt: "Event Book has a new place to start: the Pad, a canvas of plain sticky notes. Notes written in a small notation turn into slices, scenarios, screens and automations on the board when you say so."
---

Jocelyn asked: "can we do a write up of the features in eventbook on the dev log? with screenshots preferrably?" Fair. A lot has landed since the last post, and most of it is one thing: the Pad. Here is what it does and how it fits with the rest.

Every picture below is from a made-up shop basket: one small sample book that lives only in a browser.

![The Pad zoomed out: four frames of sticky notes in two columns (Add a product, Products in the basket, Remind about a full basket, Payments arrive) and a fifth frame below with a row of example notes]({{ '/assets/2026-11-26-event-book-pad/pad-overview.png' | relative_url }})

## The Pad

Event modeling usually starts on a wall with sticky notes. Event Book now has that wall. Open a book, switch from Board or Outline to **Pad**, double-click for a note and type. Drag it, recolour it, draw a frame round a few notes. Ctrl/Cmd+Z undoes. Drag on empty canvas to select an area, or switch to Pan; a phone pans with one finger and zooms with two.

The colour is the kind: blue is a command, orange an event, green a view, red a hotspot, white a plain note, grey a scenario, and two more for screens and automations. Hover a colour in the strip and a card says what it is for and how to write it, with an example you can drop onto the canvas. The **?** button opens the whole guide.

![The Automation card open under the colour strip: what an automation is, how its three lines are written, and an example note]({{ '/assets/2026-11-26-event-book-pad/pad-guide-card.png' | relative_url }})

A new model started the "Pantser" way now opens on the Pad (#352). You sketch first. You decide later what becomes a model.

## Notes are written in a notation

Nothing here is a form. A note is text, and the Pad reads it:

- A command or event is its name on the first line, then `- field` lines. A value after a field (`- basketId: B1`) makes it an example, not a definition.
- A scenario note starts with its name (a leading `★` or `*` marks a must-have), then `G:` for what happened, `W:` for the command, `T:` for the result. `T: rejected: max 10 of one product` is a refusal. On a view, `Q:` is the query and the rows under `T:` become a table.
- A screen is a low-fi window: `{ Products In Basket: productId, quantity }` shows a view, `[ Product ]` is an input, `( Add Product )` is a button that sends the command.
- A ⚙ automation says `reads Abandoned Baskets` and then `→ Send Basket Reminder`, a command, or an event it translates into.
- A red note is a question. Make chapter turns it into an open comment on the slice. A white note becomes the slice's description.
- Name an event `Payment Received (external)` and it turns yellow: another system's event.

Both the guide and the colours are the same notation the agent tools read, so a sketch typed by hand and one written by an agent look the same.

## Names and fields as you type

The Pad suggests while you type (#345, #350). Start a blue note and it offers the commands the book already has. After `G:` it offers events, after `W:` commands, after `reads` views. Open a bracket after a name and it offers that note's fields. On a phone the suggestions sit as chips above the keyboard.

![A new orange note being typed, with the text Product A and a suggestion Product Added marked EVENT under it]({{ '/assets/2026-11-26-event-book-pad/pad-autocomplete-names.png' | relative_url }})

![A new scenario note with the text W: Add Product open bracket pro, and the field productId offered under it]({{ '/assets/2026-11-26-event-book-pad/pad-autocomplete-fields.png' | relative_url }})

## Frames are slices

A frame is one slice: a command (or a view, or an automation) with its events and scenarios. Select frames and press **Make chapter…**. A sheet shows what will be made before anything is: the slices, their fields, the screen with what its inputs map to, the scenarios, the red notes that become comments. Untick what you don't want.

![The Make a chapter sheet: chapter name Basket, two slices, the command Add Product with its fields, the event Product Added, a hotspot becoming a comment, a note becoming a description, the Basket screen with its input and button mappings, and the first scenarios]({{ '/assets/2026-11-26-event-book-pad/pad-make-chapter.png' | relative_url }})

The frames keep a **✓ → Basket** chip afterwards, and if you change the sketch later the frame offers **Update chapter** to carry the change over.

![Two frames on the Pad with a green tick chip beside their names that says Basket]({{ '/assets/2026-11-26-event-book-pad/pad-chapter-chips.png' | relative_url }})

**🔗 Link copy** puts the same note in a second place. Edit one and they all change. The Basket screen above lives in two frames that way, and so does `Product Added`: it is made in one slice and read in another. Tap the 🔗 and you get a list of every place the note is, with a jump to each (#360).

An event also says who makes it. Select `Product Added` in the second frame and a button says **Made by Add Product ↗**. Tap it and the Pad goes to that slice. If no command makes the event yet, it says that instead, and when two do it warns (#351, #360).

![Product Added selected in the Products in the basket frame, with a button under the toolbar saying Made by Add Product]({{ '/assets/2026-11-26-event-book-pad/pad-made-by.png' | relative_url }})

![A sheet titled Product Added is in 2 places, listing the frame Add a product marked original and made here with a jump arrow, and Products in the basket marked here]({{ '/assets/2026-11-26-event-book-pad/pad-places.png' | relative_url }})

## Scenario strips

This one came out of a real session. Jocelyn used the Pad while event modeling with other people and found the notes-in-a-row way of working helpful, so it is now a feature (#359).

Lay example notes in a row: events with values, then a blue command with its values, then the event that should follow. Select the row and **Make scenario…** reads it from left to right as Given, When, Then.

![Four example notes in a row in a frame called Second product, all selected: Basket Opened, Product Added, Add Product and Product Added, with a Make scenario button under the toolbar]({{ '/assets/2026-11-26-event-book-pad/pad-strip.png' | relative_url }})

![The Make a scenario sheet: name Add a second product, must-have ticked, and the four lines it will write as G, G, W and T]({{ '/assets/2026-11-26-event-book-pad/pad-strip-sheet.png' | relative_url }})

The grey scenario note goes under the row, inside the frame. It works the other way too: select a scenario note and **Lay out as notes** draws it as a row of example notes again, so you can talk it through on the wall and write it back down. Each is one undo.

![A grey scenario note named Add a second product, with the example notes it was laid out as in a row below it]({{ '/assets/2026-11-26-event-book-pad/pad-strip-laid-out.png' | relative_url }})

## Copy, paste, share

Ctrl/Cmd+C, X and V work on notes and frames, and they work across books, so a slice you got right once can move to the next book (#361). **＋ Events** takes a pasted list, one event per line, and **＋ Many** does the same for any kind.

Share has a new option, **Picture of the Pad**: the whole Pad or just a selection as an image, plus a link that opens on the Pad.

![The share sheet with the Picture of the Pad tab chosen, showing a preview of all the frames]({{ '/assets/2026-11-26-event-book-pad/pad-share-picture.png' | relative_url }})

## On the board

The board learned several things from the Pad.

**Screens are edited as text.** A screen's ⋮ menu has **Edit as text**: the same box, the same suggestions, a live window and a list of what it maps. Save puts the input fields on the command and the view's fields on the screen (#354).

![The Edit screen sheet: the text of the Basket screen, the window it draws, and three mapping lines: product and quantity go to Add Product, productId and quantity come from Products In Basket]({{ '/assets/2026-11-26-event-book-pad/board-screen-as-text.png' | relative_url }})

Then Trace follows a field through the screen: Basket, Add Product, Product Added, Products In Basket.

![The board with a trace of the field quantity highlighted along four numbered stickies, and the path shown at the bottom]({{ '/assets/2026-11-26-event-book-pad/board-trace.png' | relative_url }})

**A reused sticky says where else it is.** The 🔗 2 on a card opens the list of slices it appears in, and the jump flashes the sticky it lands on (#357).

![A sheet titled Basket is in 2 slices, listing S1 Add a product as here and S2 Products in the basket with a jump arrow]({{ '/assets/2026-11-26-event-book-pad/board-places.png' | relative_url }})

**Integration events are yellow** like external ones, with the dark ↗ edge to tell them apart (#358). Here `Product Added` is published to Shipping and Shop front.

![The board's Basket chapter, with Product Added drawn yellow with a dark arrow edge and the line to Shipping and Shop front]({{ '/assets/2026-11-26-event-book-pad/board-integration-event.png' | relative_url }})

There is also a ☁ chip on the Board, Outline and Pad that says whether the book is saved to your library (#353). We left it out of the screenshots, because it needs an account.

It all works on a phone too. Here is the same chapter on a 390 px screen: the sheet before Make chapter, and the board after.

![A phone showing the Make a chapter sheet with the Add Product command and Product Added event ticked]({{ '/assets/2026-11-26-event-book-pad/phone-make-chapter.png' | relative_url }})

![A phone showing the board after Make chapter: the Basket screen, the Add Product command, the Product Added event and the scenarios]({{ '/assets/2026-11-26-event-book-pad/phone-board.png' | relative_url }})

## For agents

Your agent gets the same Pad through the [MCP server](https://github.com/jocelynenglund/event-book-mcp). `new_model` takes `sketch: true`, and `live_view` can open on the Pad, so you can watch the notes arrive. The tools are `pad_outline`, `pad_add_notes`, `pad_frame`, `pad_link_copy`, `pad_move`, `pad_remove`, `pad_rename_frame`, `pad_make_chapter` and `pad_update_chapter` (#342). Frames never overlap and notes fit inside their frame (#348), so what the agent draws is readable. The agent can sketch, you rearrange, and then it makes the chapter.

Slices now have stable refs for agents (`#k3f9q` style), so a tool call still lands on the right slice when the S-numbers shift after a move, and `retire_plan` says which built slice confirmed each planned one (#362).

`update_mcp` pulls the newest server for you: it fast-forwards your clone and tells you which commits came with it. Reconnect with `/mcp` afterwards.

## Try it

Open [eventbook.itsybit.se](https://eventbook.itsybit.se/), start a new model and press **Pad**. Double-click, type a command, and see what it offers.
