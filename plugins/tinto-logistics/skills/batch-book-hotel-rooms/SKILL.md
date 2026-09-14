---
name: batch-book-hotel-rooms
description: >
  This skill should be used when Tamara asks to "request a block of rooms
  from [hotel]," "hold rooms for [hotel] for [season]," "[tour] just sold,
  book the rooms," "convert the hotel hold for [tour]," or otherwise needs
  to either open a new seasonal hotel room hold or turn a held block into a
  sold tour's actual booking.
metadata:
  version: "0.1.0"
---

# Batch-Booking Hotel Rooms

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses (`logistics-reference.md`, "First-contact outreach: rolling, one exception"). Instead, Tamara (or Nélia) requests a block of rooms from a hotel ahead of any specific tour sale, for a whole upcoming season at once. This skill covers both halves of that process, both of which write to the production base's **`Hotel Room Inventory`** table.

## Half 1 — requesting a new hold

When Tamara asks to request or hold a new block of rooms from a hotel:

1. Confirm the hotel (Suppliers record), the season/date range, and the number of rooms she wants held.
2. Draft the request email to the hotel — same English-first, thread-aware drafting rules as `daily-supplier-communications` apply here too (check for an existing Gmail thread with this hotel before creating a new draft; draft in English regardless of the hotel's `Language Preference`, flagging translation as a follow-up if needed).
3. Create a new `Hotel Room Inventory` row: linked Hotel, the date range, room count, `Status` = **Available** (not yet sold — this is what powers the "what's left to sell" view). Don't set `Sold Tour` — there isn't one yet.
4. Confirm back to Tamara what was requested and recorded.

## Half 2 — converting a sold block

When a tour actually sells against a held block (Tamara says something like "[tour] just sold, book the rooms" or "[tour] is using the [hotel] hold"):

This has **two required steps** — doing only the first one is a real, documented gap (`logistics-reference.md`), not a smaller version of the same task:

1. **Update the `Hotel Room Inventory` row**: `Status` → **Sold**, link the `Sold Tour`. Never delete the row — it stays as historical record, same append-don't-erase convention as everywhere else in this system.
2. **Create a new `Supplier Booking Lead Times` row** for that Tour × Hotel pairing. This is the step that actually plugs the sale into the rest of the automation — without it, Room Count Reconciliation, Hotel Final Confirmation, and Final Reconfirmation will never fire for this tour's hotel booking, even though `Hotel Room Inventory` correctly shows it as sold. Set the lead time per the hotel's own confirmed `Hotel Cancellation Policy` if known (see `logistics-reference.md`); if the hotel's cancellation terms aren't confirmed yet, use the standing 45-day placeholder + Sabrina's 7-day buffer (52 days) and flag it in your confirmation back to Tamara as needing the hotel's real terms once available.

Confirm back to Tamara exactly what was updated/created in both tables — this is easy to half-do without noticing, so the confirmation should explicitly name both writes, not just the `Hotel Room Inventory` one.

## What's hers to change, and what isn't

Requesting holds and converting sold blocks are ordinary day-to-day operation — Tamara's own territory, no permission check needed. She should not change the cancellation-buffer policy (the 7-day buffer, or which lead-time rule applies to which touchpoint) — that's a system-wide rule and stays with Peter/Nina, same boundary as everywhere else in this plugin.

## Escalate, don't fix, when you see:

- A hotel with no confirmed cancellation policy and no obvious placeholder to fall back on.
- A tour that already appears to have a `Supplier Booking Lead Times` row for this hotel (possible duplicate — check before creating a second one).
- Any request that sounds like it's actually about negotiating new terms with a hotel, rather than requesting/converting a room block under existing terms — that edges toward Sales/BD, flag it rather than treating it as ordinary data entry.
