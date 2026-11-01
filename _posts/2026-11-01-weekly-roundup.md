---
layout: post
title: "EVE gets a beta tag and barrio gets a nicer hello"
date: 2026-11-01
tags: [weekly-roundup, eve, barrio]
---

A short one this time. Most of what shipped since Friday went into the EVE template, and barrio got two small fixes.

## EVE: a first beta

EVE v0.1.0-beta is out. It started with a landing page in the itsyBIT studio look, which was handed over to us to publish, and the same page got a What's new pill through a Pages workflow, like our other sites.

Event Book on the EVE site now uses the studio look too. We also changed it so that built slices fold away when a follow-up is planned, so the board doesn't keep showing work that is already done.

Two smaller things came with it. People who copy the template needed a way to get later changes, so there's now an update script, `eve/update.mjs`. And `built.json` used to change on every test run, which made for noisy diffs. It doesn't any more.

## barrio: long wishes and a new welcome

On the wish show, a long message used to overflow the note. The text now fits, the "from" stays visible, and the print takes the shape of the photo.

The share card and the default welcome now say "Dayon ta! Come on in." instead of "Tuloy po kayo".

## Reflection

**What went well:** EVE went from a template to something with a version number and a way to update it.

**What could be better:** The notes for this stretch were thin, so this post only covers what the deploy list says.

**Shipped:**
- EVE v0.1.0-beta, landing page, What's new pill, studio-look Event Book, update script
- Stable `built.json`
- barrio: long wish text fits, new welcome line
