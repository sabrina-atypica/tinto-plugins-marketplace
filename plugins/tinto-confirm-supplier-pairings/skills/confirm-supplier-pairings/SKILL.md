---
name: confirm-supplier-pairings
description: >
  This skill should be used when Tamara asks to "audit the supplier
  pairings," "review supplier assignments," "go through the tour-supplier
  pairings by destination," "confirm suppliers for [destination]," "is
  [supplier] really booked for [tour]," "check what's already been sent to
  [supplier]," "flag a lead time that looks wrong," or when
  `daily-supplier-communications` reports rows blocked on pairing
  confirmation. It should also be run once, destination by destination, as
  the very first thing Tamara does — before the `tinto-logistics` plugin's
  daily scheduler runs for the first time, not after.
metadata:
  version: "0.2.0"
---

# Confirm Supplier Pairings

`Supplier Booking Lead Times` — the table `daily-supplier-communications` reads every day to decide who to email — was built in two passes, and most of it still needs a human check before it should be treated as fact:

- **2026-09-09 bulk wire (the majority of the table):** every tour was matched to suppliers by **region only** — "this supplier operates in the same area as this tour," not "this specific supplier is booked for this specific tour." Every one of these rows has `Confidence` = "Draft estimate — needs confirming."
- **2026-09-14 itinerary pairing (a separate table, `Bookings (Confirmations)`):** real day-by-day content extracted from actual 2027 guest itinerary PDFs, tour by tour. This is much better evidence of which supplier is actually used — but it's a different table from the one the scheduler reads, and it has its own confidence tiers (some rows are a clean match, many are a best-guess or no-guess flagged row — see that table's own Notes field on each row).

This skill goes through the region-matched pairings with Tamara, destination by destination, in three phases run back to back for the same batch: **who** the supplier actually is, **what's already happened** with them for this tour, and **whether the timing looks right**. All three write directly onto the rows `daily-supplier-communications` reads, so that skill can draft safely once a pairing has been through here (see its "Gate: don't draft to an unconfirmed pairing" section).

## Why batch by destination

There are 9 destinations and roughly 28 real tours, and most tours within one destination share most of their suppliers (the Alentejo rotation, the Peloponnese circuit, etc.) — so a destination-sized batch is small enough to review in one sitting, and reviewing one destination usually resolves most of the pairings for every tour in it at once, since the same supplier↔category pairing repeats across tours. Work through destinations one at a time, in whatever order Tamara wants (the ones with the soonest tour Start Dates first is a reasonable default if she has no preference) — never try to present all 9 destinations in a single message, that's unreadable and un-answerable in one pass.

When a destination's tours share an identical set of region-matched suppliers (common, since the 2026-09-09 wire matched by region, not by tour), present each recurring pairing **once** with a per-tour breakdown only where the evidence actually differs across tours — don't repeat 10 identical rows for 10 tours that all show the same answer. Roll it up; call out the exceptions by name.

## Phase 1 — who the supplier actually is

1. **Pull what still needs review.** In production, get every `Supplier Booking Lead Times` row where `Confidence` = "Draft estimate — needs confirming" AND the linked Tour's destination (`Location`) matches, AND `Notes` does **not** already contain `[PAIRING CONFIRMED BY TAMARA` or `[PAIRING CORRECTED BY TAMARA` (already-reviewed rows are done, skip them — this is what makes the audit safely resumable across sessions and destinations).
2. **Collapse to one question per (Tour, Supplier) pair**, then roll up further across tours per the batching note above. A single pairing usually has several rows (First Contact, Final Reconfirmation, maybe Room Count Reconciliation) sharing the same Tour + Supplier — Tamara only needs to answer once per pairing, not once per row.
3. **Pull supporting evidence from `Bookings (Confirmations)`** for the same Tour, to make the question concrete instead of "is X right, yes or no":
   - If a `Bookings (Confirmations)` row for this Tour links the *same* Supplier, and its Notes don't read as an unconfirmed/no-candidate flag — say so: "the itinerary backs this up ('<Activity Description>')."
   - If a `Bookings (Confirmations)` row for this Tour, in a plausibly matching category, links a *different* Supplier — flag the conflict plainly: "the itinerary may actually point to <other supplier>, per '<Activity Description>' — worth double-checking which one is right."
   - If there's no `Bookings (Confirmations)` row for this Tour in this category at all — say so plainly ("no itinerary evidence for this one") rather than implying support that doesn't exist.
   - Treat this evidence as a prompt to help Tamara judge faster, never as grounds to auto-resolve anything yourself — both tables were built by Claude without human review, so agreement between them is a good sign, not proof. She still answers every pairing herself.
4. **Present the batch and ask her to answer each pairing**: confirm it's right, correct it (name the right supplier), or flag it as genuinely unknown (needs Sabrina/Nina, or the real supplier isn't in Airtable yet at all).
5. **Apply her answers**, per pairing, across every `Supplier Booking Lead Times` row sharing that (Tour, Supplier) group:
   - **Confirmed** → append `[PAIRING CONFIRMED BY TAMARA <today's date>]` to each row's Notes (append, never overwrite existing Notes content, same convention as everywhere else in this system).
   - **Corrected** → update the `Supplier` link on each row to the right Supplier record, then append `[PAIRING CORRECTED BY TAMARA <today's date>: was <old supplier>, now <new supplier>]`. If the right supplier has no Suppliers record at all yet, don't invent one — say so, and treat it the same way the 2026-09-14 pairing pass treated new suppliers (worth a look against `itinerary-new-suppliers-flagged.md` in case it's already listed there; otherwise flag it as a new entry for that doc). Leave the row's `Supplier` link and `Confidence` as they are until a real record exists to point it at.
   - **Unknown / needs escalation** → append `[PAIRING UNRESOLVED <today's date> — flagged for Sabrina/Nina]`. Don't change the Supplier link or Confidence. This keeps the row visibly distinct from both the reviewed-and-confirmed and the never-reviewed cases.

Only pairings that came out of Phase 1 **Confirmed or Corrected** move on to Phase 2 and 3 below in the same pass — an Unresolved pairing has no known real-world supplier yet, so there's nothing to check Gmail or timing against.

## Phase 2 — what's already happened with this supplier for this tour

Airtable's `Booking Request Status` is only reliable for what this system itself has done (drafted, and Tamara-confirmed as sent). It says nothing about outreach that happened before this system existed, or outside it entirely — and for a supplier relationship this old, that's likely to be most of it. Don't trust a blank or "Not Due Yet" status as proof nothing's happened; check.

For each pairing that came out of Phase 1 Confirmed or Corrected:

1. **Search Gmail** (`search_threads`, `to:<supplier email> OR from:<supplier email>`, loosely combined with the tour name or a date in its range — keep it loose enough to catch a thread under a slightly different subject) for any existing correspondence with this supplier. Same technique `daily-supplier-communications` uses for thread-matching before drafting.
2. **If nothing turns up**, say so plainly and leave `Booking Request Status` as it is — this is a genuinely fresh pairing as far as this system can tell.
3. **If something turns up**, don't silently trust that it's about this tour — subject-line/snippet matching is a heuristic, not proof, same caution `daily-supplier-communications` applies to its own thread-matching. Pull the thread (`get_thread`, `PLAIN_TEXT`) and surface it to Tamara plainly: name the supplier, the date of the earliest relevant message, and a short read of what it looks like ("looks like a first-contact email went out 2026-08-02, no reply yet" / "there's a full back-and-forth that reads like this is already confirmed"). Ask her what it actually means for this specific tour — already contacted, already confirmed, or unrelated to this booking.
4. **Apply her answer** to every `Supplier Booking Lead Times` row in this pairing: if she confirms it's real correspondence for this tour, update `Booking Request Status` to match (typically "Sent," or "Drafted" if it was drafted but evidently never sent) and append `[STATUS BACKFILLED BY TAMARA <today's date>, from existing Gmail thread dated <date>]` to Notes — distinct from the pairing markers in Phase 1, so it's clear this reflects real-world status, not just a confirmed identity. If she says the thread is unrelated, leave the row alone and say so.
5. **Ask once per destination batch, not once per pairing**, whether anything outside Gmail should feed into this too — a spreadsheet, a WhatsApp log, notes Peter or Nina kept — since Gmail only shows what happened to go through that channel. If she has something, treat what she tells you the same way as a confirmed Gmail finding: update status, note the source plainly in Notes rather than implying it came from Gmail when it didn't.

## Phase 3 — does the timing look right (flag only, never change)

Lead-time windows are Peter/Nina's call, not Tamara's — `module1-permission-split-draft.md` and `logistics-reference.md` are both explicit about this, and `daily-supplier-communications` already treats an odd lead time as something to escalate, not patch. This phase exists so Tamara can raise a concern with real context in front of her, not to give her editing authority over the policy.

For each pairing confirmed or corrected in Phase 1, state its current `Booking Lead Time (Days Before Tour Start)` value plainly (e.g. "RSI, Transport, currently set to 210 days") and ask if it looks obviously wrong for this specific supplier — not "do you want to change it," just "does this look right to you." Most of the time the answer is nothing to flag; don't force a response out of her for every single pairing if she'd rather wave the whole destination through.

If she flags one: **do not change `Booking Lead Time (Days Before Tour Start)` or any other timing field.** Append `[TIMING FLAGGED BY TAMARA <today's date>: <her reasoning>]` to the row's Notes instead, and say plainly in the destination close-out that this needs a Peter/Nina decision, not a Claude one.

## Closing out a destination batch

State plainly, across all three phases: how many pairings were confirmed / corrected / left unresolved (Phase 1), how many had their status backfilled from existing correspondence vs. genuinely fresh (Phase 2), and how many timing flags were raised for Peter/Nina (Phase 3). Say that `daily-supplier-communications` will now draft normally for the confirmed/corrected pairings next time they're due. Offer to move on to the next destination or stop here — since Phase 1's first step only pulls unreviewed rows, picking this back up later (same destination or a different one) just works, nothing needs to be remembered between sessions.

## What's hers to change, and what isn't

Confirming or correcting which real-world supplier a row points to, and backfilling real-world contact status, are both ordinary day-to-day operation — exactly the kind of judgment call Tamara is best placed to make, no permission check needed. She should not use this skill to invent a lead-time rule for a supplier category that doesn't have one yet (Wine Bar, Cooking Class, Entertainment, Olive Producer, Activity/Tour, Winery/Restaurant combo, Cultural Site — see `logistics-reference.md`), and she cannot change an existing lead-time value through this skill at all, even one she's confident is wrong — see Phase 3. Both are Peter/Nina system-wide decisions; flag them instead of resolving them.

## Escalate, don't fix, when you see:

- A pairing where neither the region-matched guess nor the itinerary evidence gives a confident answer, and Tamara herself isn't sure — that's exactly what "Unknown" in Phase 1 is for; don't push her to guess.
- A Tour with no `Bookings (Confirmations)` rows at all and no obvious regional candidate either — say so, don't leave it silently unaddressed.
- Any correction that would require creating a brand-new Suppliers record — that's real data entry, not a pairing fix; point her at `itinerary-new-suppliers-flagged.md` or flag it as a new addition to that doc rather than creating the record yourself mid-audit.
- A Gmail thread that reads as genuinely ambiguous (Phase 2) — when in doubt, don't guess what it means; ask, or leave the row's status alone and say why.
- Any lead time Tamara flags as wrong (Phase 3) — record it, don't act on it.
