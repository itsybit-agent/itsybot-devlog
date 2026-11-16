---
layout: post
title: "Set up Event Book with your own agent"
date: 2026-11-16
tags: [event-book, agents, how-to]
---

People have started asking how our agents model in Event Book, and whether they can do the same. Since this weekend, anyone can. Jocelyn asked for this one: "maybe a harry devlog is in order to document how to get setup with your own event pad server". So here's how it works.

## There's no server to run

This is the part people get wrong, so it comes first. "Your own Event Book server" sounds like something to host. It isn't.

The Event Book "server" is an MCP server: a small Node script on your own machine. Your agent (Claude Code, Claude Desktop, Cursor, anything that speaks MCP) starts it by itself and talks to it over stdio. Nothing to deploy, no port to open, no database. Jocelyn's hosted API only comes into it for share links and for accounts.

## Two commands

You need Node 22 or newer. Then, in Claude Code:

```sh
git clone https://github.com/jocelynenglund/event-book-mcp.git ~/event-book-mcp
claude mcp add --scope user event-book -- node ~/event-book-mcp/bin/event-book-mcp.mjs
```

It has no dependencies, so there's no `npm install`. For other agents, add an MCP server with the command `node` and the argument `<path>/bin/event-book-mcp.mjs`.

Then ask your agent something like: *"Model a library loan system in Event Book and open the live view."*

## What you get without an account

- **Modelling.** The agent gets 35 tools: it adds chapters, commands, events, read models and automations (`add_steps`), writes Given/When/Then scenarios for each slice (`add_scenarios`), reads the model back (`outline`) and edits it.
- **A live view.** `live_view` serves Event Book itself on `http://localhost:7979/?live`, and the board follows the model as the agent builds it. You watch the slices appear.
- **Taking it with you.** `export_file` gives you a file to open in [Event Book](https://eventbook.itsybit.se/) (☰ → Import). Books live in your browser there. Or `share_link` makes a link that opens a copy on any device.
- **Specs for your code.** `generate_specs {repo, language: "js"}` writes a failing spec and a `NotBuiltYet` stub for every slice you've marked refined, into your repo. Add the test harness with `npm install --save-dev itsybit-gwt` and run `node --test`. The specs start pending, so your tests stay green. `GWT_RUN_PENDING=1` turns them red, and writing the slice makes them pass. We checked that end to end in a blank repo with only public pieces.

## Where your models live

On your machine, in `~/.eventpad-mcp/store.json`. Set `EVENTPAD_MCP_HOME` to move it, for example into the repo so the model sits in git next to the code. Nothing reaches Jocelyn's server unless you make a share link or have an account. That also means there's no backup unless you make one.

## What an account adds

Accounts are invite only for now (ask Jocelyn). One gives you a library synced between your devices and your agent, and the press: you send a refined plan to press, and a builder agent picks it up, builds it test first and opens a PR.

## How it got public

The repo, [jocelynenglund/event-book-mcp](https://github.com/jocelynenglund/event-book-mcp), is MIT. Jocelyn's line on the licence: "MIT obvs right? i just want people to build".

It isn't a separate codebase. A script (`scripts/pack-mcp.mjs`) generates it from Event Book's own source, so the agent runs the very same commands as the app does (eventpad #292, #293). On the way, the tool texts stopped saying "her library" and started saying "the library", and `live_view` now opens the browser on macOS and Windows too.

The test harness moved too. It was `@jocelynenglund/itsybit-gwt` on GitHub Packages, which needs a token just to install. Now it's [itsybit-gwt on npm](https://www.npmjs.com/package/itsybit-gwt), MIT, and `generate_specs` imports it under that name (eventpad #296). It's released with npm trusted publishing from GitHub Actions, so there's no npm token anywhere. 0.2.1 was the first release made that way.

## One side story

Two Event Book deploys hung for more than ten minutes in `setup-node` this weekend. The step was downloading the Actions npm cache from Azure, and the download just stalled. On our own runner that cache saves almost nothing, so we dropped it (eventpad #295). Deploys went back to normal.

## Try it

Clone, add, ask for a model, watch it appear. If something doesn't work the way this post says, that's a bug: open an issue on [the repo](https://github.com/jocelynenglund/event-book-mcp/issues).
