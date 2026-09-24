---
name: batch-book-hotel-rooms
description: >
  This skill should be used when Tamara asks to "request a block of rooms
  from [hotel]," "hold rooms for [hotel] for [season]," "the hotel got back
  to us on [request]," "[tour] just sold, book the rooms," "convert the
  hotel hold for [tour]," or otherwise needs to open a new seasonal hotel
  room hold, record a hotel's reply to one, or turn a confirmed block into
  a sold tour's actual booking.
metadata:
  version: "0.5.0"
---

# Batch-Booking Hotel Rooms

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses (`logistics-reference.md`, "First-contact outreach: rolling, one exception"). Instead, Tamara (or Nélia) requests a block of rooms from a hotel ahead of any specific tour sale, for a whole upcoming season at once — before any client, winery, or tour exists. This skill covers all three stages of that process, all of which write to the production base's **`Hotel Room Inventory`** table: requesting a hold, recording the hotel's reply, and converting a confirmed hold into a sold tour's actual booking (which also bridges into **`Room Blocks`** — see Half 2). A sibling skill, `check-hotel-replies`, runs on a schedule to do the watching-the-inbox part of Half 1.5 for you — it reads a hotel's reply and proposes what to record, but never writes to this table itself; see that skill for the mechanism, and its own section below for what changes here to support it.

`Hotel Room Inventory.Status` tracks every requested combo of hotel × week through four stages: **Requested** (asked the hotel, awaiting reply) → **Available** (hotel confirmed it, held, sellable to clients/affiliates) → **Sold** (a specific tour bought it); or **Requested** → **Not available** (hotel couldn't hold it). Available is what "preemptively booked" means in this system — Sold is a separate, later stage, not a synonym for it.

## Half 1 — requesting a new hold

When Tamara asks to request or hold a new block of rooms from a hotel:

1. Confirm the hotel (Suppliers record), the specific week or date block being requested (`Week Start Date` / `Week End Date` — a season-long rate offer is usually quoted for several date blocks at once, so confirm which one(s) she actually wants requested now), the room count, and the room category (`Room Type` — e.g. "Superior, Castle view, double occupancy" — read the hotel's rate offer in `Suppliers.Notes / Specialties` for the real category names rather than inventing one).
2. Draft the request email to the hotel — same English-first, thread-aware drafting rules as `daily-supplier-communications` apply here too (check for an existing Gmail thread with this hotel before creating a new draft; draft in English regardless of the hotel's `Language Preference`, flagging translation as a follow-up if needed). Note the Gmail thread ID this draft belongs to (or will belong to, once sent) — it's needed in the next step.
3. Create a new `Hotel Room Inventory` row: linked Hotel, `Week Start Date`/`Week End Date`, `Rooms Held`, `Room Type`, `Status` = **Requested** (asked, not yet confirmed by the hotel — this is what makes it show up as awaiting reply rather than sellable), `Requested Date` = today (when the request was actually sent). Don't set `Sold Tour` or `Cancellation Deadline` — there's no confirmed hold yet to attach either to; those come once the hotel replies (Half 1.5). Write a short human-readable narrative into `Notes` (e.g. "Requested 20 rooms, Superior Castle view, for 12–19 Jun 2027, per [hotel]'s season rate offer"), then append a `[DRAFTED <date> THREAD:<gmail thread id>]` marker — same append-don't-overwrite Notes convention used everywhere else in the base, so anyone reading the record later understands the ask without cross-referencing the request email, and so `check-hotel-replies` (below) knows which thread to watch. If one request covers several weeks at once, put the same thread marker on every row it produced — a single hotel reply may confirm some and decline others.
4. Confirm back to Tamara what was requested and recorded, including the room category, and that it's logged as **Requested** pending the hotel's reply.

## Half 1.5 — recording the hotel's reply

When Tamara reports back what the hotel said about a pending request (e.g. "Herdade X confirmed the 12 and 19 June weeks but not 26 June," or "the hotel came back, we're good on all of it") — or when she confirms a proposal that `check-hotel-replies` already surfaced ("yes, go ahead and record that"), treat her confirmation the same way, using the reading it proposed:

1. Find the matching `Hotel Room Inventory` row(s) — same Hotel, `Status` = **Requested**, and the week(s) Tamara is referring to. If nothing in Requested status matches (wrong hotel, dates don't line up, or there's no pending request for this hotel at all), don't guess or create a look-alike row — ask her which request this reply belongs to.
2. For each week the hotel confirmed: `Status` → **Available**, and set `Cancellation Deadline` from the hotel's own `Hotel Cancellation Policy` field — read the free-cancellation window stated there (e.g. "free cancellation up to 45 days before arrival") and set the deadline to `Week Start Date` minus that many days. If `Cancellation Policy Confirmed Date` is blank or the policy text doesn't give a clear day count, don't guess — leave `Cancellation Deadline` blank and flag it in your confirmation back to Tamara as needing the hotel's confirmed cancellation terms. (This field is what Room Count Reconciliation acts on to release unsold rooms before the hotel's free-cancellation window closes — see `logistics-reference.md` — so a blank or wrong deadline here is a real gap downstream, not just a missing detail on this record.) Append a dated Notes marker, e.g. `[CONFIRMED <date>] Hotel confirmed this week available.`
3. For each week the hotel declined: `Status` → **Not available**. Never delete the row — it stays as historical record, same append-don't-erase convention as everywhere else in this system. Append `[DECLINED <date>] Hotel could not hold this week.` to Notes rather than overwriting anything already there. This is a dead end for that hotel × week combo, not something to quietly retry.
4. If the hotel's reply changes the room count or category from what was originally requested (e.g. asked for 20, offered 15), update `Rooms Held` / `Room Type` to match what's actually confirmed and note the discrepancy in Notes — don't silently leave the original ask standing as if it were confirmed.
5. Confirm back to Tamara which weeks are now **Available** (and therefore ready to start offering to clients/affiliates one by one) and which are **Not available**, and flag any `Cancellation Deadline` that's still unset.

## Half 2 — converting a sold block

When a client or affiliate actually buys against an **Available** held block (Tamara says something like "[tour] just sold, book the rooms" or "[tour] is using the [hotel] hold"):

This has **three required steps** — doing only one or two of them is a real, documented gap (`logistics-reference.md`), not a smaller version of the same task:

1. **Update the `Hotel Room Inventory` row**: confirm it's currently `Status` = **Available** — if it's still **Requested**, the hotel never actually confirmed this week; don't mark it Sold on top of an unconfirmed hold, ask Tamara to confirm the hotel's reply first (Half 1.5). Then `Status` → **Sold**, link the `Sold Tour`. Never delete the row — it stays as historical record. Append a dated marker to `Notes` rather than overwriting what's already there — e.g. `[SOLD <date>] Converted to <Tour>` — matching the `[CONFIRMED ...]` / `[DECLINED ...]` marker convention above.

   **Ask explicitly how many of the `Rooms Held` are complimentary** — given free to the tour organizer or affiliate who sold the tour, rather than sold to end guests (Tinto's standing practice is at least one comped room per sold tour, but never assume the number — ask every time, and treat 0 as a valid, explicit answer rather than skipping the question). Record it in `Complimentary Rooms` on this row.

2. **Bridge into `Room Blocks`** — a one-time, one-directional handoff at the moment of this conversion, not a standing sync (`Room Blocks` and `Hotel Room Inventory` aren't linked; they're allowed to diverge afterward — e.g. a later `Room Blocks` correction, or a hold that gets partly released back to the hotel — without that being an error to reconcile).
   - Compute the net sellable rooms: `Rooms Held` − `Complimentary Rooms`. This net number, not the gross `Rooms Held`, is what goes into `Room Blocks.Capacity`, so that table always reflects rooms actually available for sale to end customers.
   - Check whether a `Room Blocks` row already exists for this Tour first — some tours run a sequence of hotel legs sharing one `Room Blocks` record rather than one per hotel (e.g. Rappahannock Cellars, the Puglia "Tinto" tours). If one exists for this Tour:
     - Append this hotel to the existing `Property/Hotel` text (it's a plain text field, not linked — e.g. "DoubleTree Trieste · Occidental Ljubljana (sequence)") rather than overwriting what's there.
     - Add this leg's net rooms to the existing `Capacity` only when the legs are sequential (same trip, different nights) so the block's total sellable capacity is still one meaningful number; if it's not obviously sequential, or the resulting total looks inconsistent with what's already recorded, flag the mismatch to Tamara rather than silently summing or overwriting.
   - If no `Room Blocks` row exists yet for this Tour, create one: `Block Name`, `Property/Hotel` = the hotel name, `Capacity` = the net figure, `Tour` linked, and a `Notes` line spelling out the math (e.g. "13 rooms held at [hotel], 1 complimentary, 12 net capacity — bridged from Hotel Room Inventory on [date]") so the source of the number is traceable later.
3. **Create or update the `Supplier Booking Lead Times` rows for that Tour × Hotel pairing — there are three, not one.** A hotel pairing tracks three separate touchpoints, each its own row, `Label`led `<Tour> — <Hotel> (<Touchpoint>)`, `Supplier Category` = Hotel:
   - **Rooming List** — `Booking Lead Time (Days Before Tour Start)` = 7 (the standard convention used throughout the base).
   - **Room Count Reconciliation** — the step that actually plugs the sale into the rest of the automation; without it, Room Count Reconciliation will never fire for this tour's hotel booking, even though `Hotel Room Inventory` correctly shows it as sold. Set the lead time per the hotel's own confirmed `Hotel Cancellation Policy` if known (see `logistics-reference.md`); if the hotel's cancellation terms aren't confirmed yet, use the standing 45-day placeholder + Sabrina's 7-day buffer (52 days), set `Confidence` = "Draft estimate — needs confirming," and flag it in your confirmation back to Tamara as needing the hotel's real terms once available.
   - **Final Confirmation** — leave `Booking Lead Time (Days Before Tour Start)` blank; this touchpoint triggers at 70% tour capacity, not on a day count (`logistics-reference.md`, "Hotel Final Confirmation").

   **Before creating anything, check whether these rows already exist for this Tour + Hotel pairing** — this is the normal case, not a rare one: `confirm-supplier-pairings` routinely pre-seeds region-matched draft rows (`Confidence` = "Draft estimate — needs confirming") for a tour's likely hotel well before it actually sells. If draft rows already exist for this Tour and the hotel that was actually held (or a hotel close enough that Tamara confirms it's the same one), **update them in place** — correct the `Supplier` link if the draft guessed a different hotel, fill in real lead-time values, and leave `Confidence`/`Booking Request Status` for Tamara or `confirm-supplier-pairings` to promote once confirmed — rather than creating a second, duplicate set of three rows. Only treat it as a genuine duplicate (escalate, don't silently create a second row) when a row already exists for this exact hotel with `Confidence` already "Confirmed by Peter/Nina (internal process)" or a populated `Booking Request Status` beyond "Not Due Yet" — that's a sign this pairing was already converted once.

Confirm back to Tamara exactly what was updated/created — the `Hotel Room Inventory` write (including the complimentary-room count), the `Room Blocks` write (including the net capacity and whether it was a new block or an append to an existing sequence), and all three `Supplier Booking Lead Times` rows (or which of the three were updated versus newly created) — not just a generic "done." This is easy to half-do without noticing.

## What's hers to change, and what isn't

Requesting holds, recording hotel replies, and converting sold blocks are ordinary day-to-day operation — Tamara's own territory, no permission check needed. She should not change the cancellation-buffer policy (the 7-day buffer, or which lead-time rule applies to which touchpoint) — that's a system-wide rule and stays with Peter/Nina, same boundary as `daily-supplier-communications` and every other Tamara-facing skill in this marketplace.

**Out of scope: `Supplier Payments`.** This skill never creates or updates rows in the `Supplier Payments` table. Logging a hotel's payment obligation is Finance's territory (Peter/Nina), not something that happens automatically off a `Hotel Room Inventory` or `Supplier Booking Lead Times` write — per the org-wide money rule (`finance-reference.md`), anything touching supplier payments needs explicit confirmation before executing. That holds even once a block converts to Sold; don't treat the connection as implied work for this skill.

## Escalate, don't fix, when you see:

- A hotel with no confirmed cancellation policy and no obvious placeholder to fall back on — for a hold just confirmed by the hotel (Half 1.5's `Cancellation Deadline`) as much as for a sold-block conversion's Room Count Reconciliation lead time.
- A `Hotel Room Inventory` row Tamara says sold that's still `Status` = Requested, not Available — the hotel's confirmation was never recorded (or never happened). Get that resolved via Half 1.5 before converting.
- A `Room Blocks.Capacity` figure for an existing sequence that doesn't add up with what this hotel leg should contribute — don't silently sum or overwrite it.
- A `Supplier Booking Lead Times` row for this Tour and Hotel that's already past the draft stage (`Confidence` = "Confirmed by Peter/Nina (internal process)," or a `Booking Request Status` beyond "Not Due Yet") — a real possible duplicate, not the ordinary pre-seeded-draft case above.
- Any request that sounds like it's actually about negotiating new terms with a hotel, rather than requesting/converting a room block under existing terms — that edges toward Sales/BD, flag it rather than treating it as ordinary data entry.
