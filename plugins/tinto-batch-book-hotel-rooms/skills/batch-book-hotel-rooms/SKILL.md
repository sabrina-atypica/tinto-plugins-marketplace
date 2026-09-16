---
name: batch-book-hotel-rooms
description: >
  This skill should be used when Tamara asks to "request a block of rooms
  from [hotel]," "hold rooms for [hotel] for [season]," "[tour] just sold,
  book the rooms," "convert the hotel hold for [tour]," or otherwise needs
  to either open a new seasonal hotel room hold or turn a held block into a
  sold tour's actual booking.
metadata:
  version: "0.3.0"
---

# Batch-Booking Hotel Rooms

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses (`logistics-reference.md`, "First-contact outreach: rolling, one exception"). Instead, Tamara (or Nélia) requests a block of rooms from a hotel ahead of any specific tour sale, for a whole upcoming season at once. This skill covers both halves of that process, both of which write to the production base's **`Hotel Room Inventory`** table.

## Half 1 — requesting a new hold

When Tamara asks to request or hold a new block of rooms from a hotel:

1. Confirm the hotel (Suppliers record), the specific week or date block being held (`Week Start Date` / `Week End Date` — a season-long rate offer is usually quoted for several date blocks at once, so confirm which one(s) she actually wants held now), the room count, and the room category (`Room Type` — e.g. "Superior, Castle view, double occupancy" — read the hotel's rate offer in `Suppliers.Notes / Specialties` for the real category names rather than inventing one).
2. Draft the request email to the hotel — same English-first, thread-aware drafting rules as `daily-supplier-communications` apply here too (check for an existing Gmail thread with this hotel before creating a new draft; draft in English regardless of the hotel's `Language Preference`, flagging translation as a follow-up if needed).
3. Create a new `Hotel Room Inventory` row: linked Hotel, `Week Start Date`/`Week End Date`, `Rooms Held`, `Room Type`, `Status` = **Available** (not yet sold — this is what powers the "what's left to sell" view), `Requested Date` = today (when the request was actually sent, not when the hold is later confirmed). Don't set `Sold Tour` — there isn't one yet. Write a short human-readable narrative into `Notes` (e.g. "Requested 20 rooms, Superior Castle view, for 12–19 Jun 2027, per [hotel]'s season rate offer") — same append-don't-overwrite Notes convention used everywhere else in the base, so anyone reading the record later understands the hold without cross-referencing the request email.
4. **Set `Cancellation Deadline`** from the hotel's own `Hotel Cancellation Policy` field: read the free-cancellation window stated there (e.g. "free cancellation up to 45 days before arrival") and set the deadline to `Week Start Date` minus that many days. If `Cancellation Policy Confirmed Date` is blank or the policy text doesn't give a clear day count, don't guess — leave `Cancellation Deadline` blank and flag it in your confirmation back to Tamara as needing the hotel's confirmed cancellation terms. (This field is what Room Count Reconciliation acts on to release unsold rooms before the hotel's free-cancellation window closes — see `logistics-reference.md` — so a blank or wrong deadline here is a real gap downstream, not just a missing detail on this record.)
5. Confirm back to Tamara what was requested and recorded, including the room category and the cancellation deadline (or the fact that it's still unknown).

## Half 2 — converting a sold block

When a tour actually sells against a held block (Tamara says something like "[tour] just sold, book the rooms" or "[tour] is using the [hotel] hold"):

This has **two required steps** — doing only the first one is a real, documented gap (`logistics-reference.md`), not a smaller version of the same task:

1. **Update the `Hotel Room Inventory` row**: `Status` → **Sold**, link the `Sold Tour`. Never delete the row — it stays as historical record, same append-don't-erase convention as everywhere else in this system. Append a dated marker to `Notes` rather than overwriting what's already there — e.g. `[SOLD <date>] Converted to <Tour>` — matching the `[DRAFTED ...]` / `[SENT ...]` marker convention used elsewhere in the base.
2. **Create or update the `Supplier Booking Lead Times` rows for that Tour × Hotel pairing — there are three, not one.** A hotel pairing tracks three separate touchpoints, each its own row, `Label`led `<Tour> — <Hotel> (<Touchpoint>)`, `Supplier Category` = Hotel:
   - **Rooming List** — `Booking Lead Time (Days Before Tour Start)` = 7 (the standard convention used throughout the base).
   - **Room Count Reconciliation** — the step that actually plugs the sale into the rest of the automation; without it, Room Count Reconciliation will never fire for this tour's hotel booking, even though `Hotel Room Inventory` correctly shows it as sold. Set the lead time per the hotel's own confirmed `Hotel Cancellation Policy` if known (see `logistics-reference.md`); if the hotel's cancellation terms aren't confirmed yet, use the standing 45-day placeholder + Sabrina's 7-day buffer (52 days), set `Confidence` = "Draft estimate — needs confirming," and flag it in your confirmation back to Tamara as needing the hotel's real terms once available.
   - **Final Confirmation** — leave `Booking Lead Time (Days Before Tour Start)` blank; this touchpoint triggers at 70% tour capacity, not on a day count (`logistics-reference.md`, "Hotel Final Confirmation").

   **Before creating anything, check whether these rows already exist for this Tour + Hotel pairing** — this is the normal case, not a rare one: `confirm-supplier-pairings` routinely pre-seeds region-matched draft rows (`Confidence` = "Draft estimate — needs confirming") for a tour's likely hotel well before it actually sells. If draft rows already exist for this Tour and the hotel that was actually held (or a hotel close enough that Tamara confirms it's the same one), **update them in place** — correct the `Supplier` link if the draft guessed a different hotel, fill in real lead-time values, and leave `Confidence`/`Booking Request Status` for Tamara or `confirm-supplier-pairings` to promote once confirmed — rather than creating a second, duplicate set of three rows. Only treat it as a genuine duplicate (escalate, don't silently create a second row) when a row already exists for this exact hotel with `Confidence` already "Confirmed by Peter/Nina (internal process)" or a populated `Booking Request Status` beyond "Not Due Yet" — that's a sign this pairing was already converted once.

Confirm back to Tamara exactly what was updated/created — name the `Hotel Room Inventory` write and all three `Supplier Booking Lead Times` rows (or which of the three were updated versus newly created), not just a generic "done." This is easy to half-do without noticing.

## What's hers to change, and what isn't

Requesting holds and converting sold blocks are ordinary day-to-day operation — Tamara's own territory, no permission check needed. She should not change the cancellation-buffer policy (the 7-day buffer, or which lead-time rule applies to which touchpoint) — that's a system-wide rule and stays with Peter/Nina, same boundary as `daily-supplier-communications` and every other Tamara-facing skill in this marketplace.

**Out of scope: `Supplier Payments`.** This skill never creates or updates rows in the `Supplier Payments` table. Logging a hotel's payment obligation is Finance's territory (Peter/Nina), not something that happens automatically off a `Hotel Room Inventory` or `Supplier Booking Lead Times` write — per the org-wide money rule (`finance-reference.md`), anything touching supplier payments needs explicit confirmation before executing. That holds even once a block converts to Sold; don't treat the connection as implied work for this skill.

## Escalate, don't fix, when you see:

- A hotel with no confirmed cancellation policy and no obvious placeholder to fall back on — for a new hold (Half 1's `Cancellation Deadline`) as much as for a sold-block conversion's Room Count Reconciliation lead time.
- A `Supplier Booking Lead Times` row for this Tour and Hotel that's already past the draft stage (`Confidence` = "Confirmed by Peter/Nina (internal process)," or a `Booking Request Status` beyond "Not Due Yet") — a real possible duplicate, not the ordinary pre-seeded-draft case above.
- Any request that sounds like it's actually about negotiating new terms with a hotel, rather than requesting/converting a room block under existing terms — that edges toward Sales/BD, flag it rather than treating it as ordinary data entry.
