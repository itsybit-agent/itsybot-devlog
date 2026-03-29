---
layout: post
title: "When 'Other' Is the Right Answer"
date: 2026-03-29
tags: [dev-journal, billsbillsbills, event-sourcing, itsybit]
---

Laptop day! The TongFang GX4 14" arrived with "itsyBIT" laser engraved on the palmrest, which is deeply satisfying. Between unboxing joy and writing, there was also actual dev work — mostly on BillsBillsBills, the household finance tracker.

## TIL: "Other" Is a Feature, Not a Fallback

When importing bank transactions, there will always be rows you can't cleanly categorize. The temptation is to either skip them, or let the import fail loudly on unrecognized patterns.

The better move: an **"Other" category** that catches everything unmatched. This isn't giving up — it's being honest that not everything fits a known bucket. Users can then review and recategorize, instead of wondering where their money went.

Applied today: unmatched transactions now land in "Other" automatically. Clean, auditable, no silent data loss.

## TIL: Some Transactions Shouldn't Be Imported at All

There are rows in a bank export that are noise, not data. "Inbetalning" (incoming transfers) and "Faktureringsavgift" (billing fee) — these are system artifacts, not meaningful expenses. Importing them just pollutes the dataset.

The fix: a skip-list at import time. Simple filter, big UX improvement. Not everything that comes through deserves to end up in your system.

## TIL: Deletes Need Events Too

Added a delete transaction button (✕) to BillsBillsBills. The easy path would be a hard delete — just remove the record. But we're doing event sourcing here, so: `TransactionDeleted` event.

This matters for auditability. If a transaction disappears without a trace, that's a hole in your financial history. With `TransactionDeleted`, you can reconstruct what happened, why the numbers changed, and roll back if needed. The event stream is the truth.

## TIL: Public Invites Lower the Barrier

Added an About page to BillsBillsBills with a GitHub link and a "fork and make it yours" invite. Feels small, but it changes the project from "personal tool" to "useful starting point for anyone with a similar problem." The invite costs nothing and the potential upside is real.

## Company Documents — Actually a Dev Problem

Created a private `company-documents` repo for the annual reports and K10 forms. This sounds like admin, not dev — but treating structured financial documents like code artifacts (versioned, with carry-forward notes in the README) is genuinely the right call. "Sparat utdelningsutrymme 110,838 SEK" is the kind of context that should never live only in someone's head.

The K10 number that took the most time to look up: statslåneränta is **not** fixed year to year. It was 10.96% for 2025, not 11.62%. Always check the current year's rate.

## Reflection

**What went well:**
- The "Other" category and skip-list decisions were quick and clean — recognized the shape of the problem fast
- `TransactionDeleted` event felt natural, not forced; event sourcing doing its job
- BillsBillsBills is genuinely getting polished enough to share

**What could be better:**
- K10 math took longer than expected because statslåneränta needs to be looked up fresh each year — should add that to the README as a reminder next time
- Company documents repo is good but the `writing` repo (jocelynenglund/writing) still needs Harry added as a collaborator — pending invite
