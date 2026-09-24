# Tinto Batch Book Hotel Rooms

Requests a new seasonal hotel room hold, checks for and interprets the hotel's reply, and converts a confirmed block into a sold tour's actual booking.

## Overview

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses. Before any client, winery, or tour exists, a block of rooms is requested from a hotel for a whole upcoming season at once. This plugin covers all of that process, writing to the production base's `Hotel Room Inventory` table, whose `Status` field tracks each hotel × week combo through **Requested** (asked, awaiting reply) → **Available** (hotel confirmed, held, sellable) → **Sold** (bought by a specific tour), or **Requested** → **Not available** (hotel declined):

1. **Requesting a hold** — creates the `Hotel Room Inventory` row as Requested, tagged with the Gmail thread the request went out on.
2. **Checking for the hotel's reply** — a scheduled skill watches that thread, reads what the hotel actually said, and proposes the resulting update; nothing writes to Airtable until Tamara confirms it.
3. **Recording the reply** — flips confirmed weeks to Available (setting the `Cancellation Deadline` from the hotel's own policy) and declined weeks to Not available, whether triggered by Tamara paraphrasing the email herself or by confirming the scheduled check's proposal.
4. **Converting a sold block** — flips the row to Sold, asks how many of the held rooms are complimentary (given to the tour organizer/affiliate rather than sold to end guests), bridges the net sellable rooms into a `Room Blocks` row for the tour, and creates or updates three `Supplier Booking Lead Times` rows for the Tour × Hotel pairing (Rooming List, Room Count Reconciliation, and Hotel Final Confirmation) — the step that actually plugs the sale into the rest of the automation.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who manages hotel blocks (typically Tamara, sometimes Nélia) — not gated behind Tamara's restricted daily-cycle plugin.

**Reviewed and rebuilt against live production schema and data, 2026-09-16.** **Updated 2026-09-16 (v0.3.0):** both halves write a human-readable `Notes` narrative on the `Hotel Room Inventory` row (append-don't-overwrite, matching the base's `[DRAFTED ...]`/`[SENT ...]` marker convention), and the scope boundary says explicitly that this skill never touches `Supplier Payments`. **Updated 2026-09-24 (v0.4.0):** `Hotel Room Inventory.Status` expanded from two choices (Available/Sold) to four (Requested/Available/Not available/Sold) to accurately track every requested hotel × week combo from first ask through to sale — matching the real Tinto process of batch-requesting rooms speculatively, long before any tour exists. This added a new "Half 1.5" step for recording the hotel's reply, and Half 2 now also bridges the sale into `Room Blocks`: it asks explicitly how many rooms are complimentary and writes only the net sellable figure (`Rooms Held` minus complimentary) into `Room Blocks.Capacity`, appending to an existing multi-hotel-sequence block rather than creating a duplicate where one applies. This is a one-time handoff at the moment of sale, not a live sync — `Room Blocks` and `Hotel Room Inventory` aren't linked and may legitimately diverge afterward. **Updated 2026-09-24 (v0.5.0):** added `check-hotel-replies`, a scheduled skill that reads a hotel's reply itself instead of Tamara having to relay it — but it only ever proposes the resulting Status update; the actual write still goes through Half 1.5, unchanged, once she confirms. Half 1 now tags each request with a `[DRAFTED <date> THREAD:<id>]` Notes marker so the new skill knows which Gmail thread to watch.

## Components

| Skill | Purpose |
|---|---|
| `batch-book-hotel-rooms` | Half 1: requesting a new room hold from a hotel (drafts the request, records `Hotel Room Inventory` as Requested with room type, requested date, and the Gmail thread it's tagged to). Half 1.5: recording the hotel's reply (flips confirmed weeks to Available with a policy-derived `Cancellation Deadline`, declined weeks to Not available) — triggered either by Tamara relaying the reply herself, or by her confirming a proposal from `check-hotel-replies`. Half 2: converting a sold block (updates `Hotel Room Inventory` to Sold with a complimentary-room count, bridges the net sellable rooms into a `Room Blocks` row for the tour, then creates or updates the pairing's three `Supplier Booking Lead Times` rows — Rooming List, Room Count Reconciliation, Hotel Final Confirmation — most of which already exist as region-matched drafts from `confirm-supplier-pairings` by the time a tour sells). |
| `check-hotel-replies` | A scheduled check that reads Gmail threads on pending Requested rows, works out what the hotel actually said, and proposes the resulting update in a plain-language ping — never writes to Airtable itself. Waits for Tamara's yes (or correction) before Half 1.5 actually applies it. |

No agents or hooks — the skills' own escalation guidance covers the judgment calls (no confirmed cancellation policy, an unconfirmed hold being treated as sold, a multi-leg `Room Blocks` capacity that doesn't add up, a genuine duplicate lead-time row versus an expected pre-seeded draft, a hotel reply that doesn't map cleanly onto what was requested, anything that edges toward Sales/BD negotiation).

**Note on Final Reconfirmation:** the 28-day, calendar-independent "Final Reconfirmation" touchpoint documented in `logistics-reference.md` for Hotel/Winery/Restaurant/Transport currently has no rows for Hotel at all in production (only Winery/Restaurant/Transport got them in the 2026-09-09 bulk wire) — a known, unresolved data gap outside this skill's scope, not something this skill invents or silently patches.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. `batch-book-hotel-rooms` itself needs no scheduled task — it's run on demand, whenever a hold is requested, a reply is being recorded, or a block sells.

`check-hotel-replies` does need a scheduled task, since its whole point is to check Gmail without being asked. Create it with the Claude Code Remote scheduling tools (`create_trigger`) from **Tamara's own account**, after this plugin is installed there — never local cron, since that wouldn't survive the session ending, same standing rule as `tinto-logistics`'s daily scheduler. It should fire once a day; turn on push notifications so a proposal actually reaches her rather than sitting unread in a chat she hasn't opened. This one-time setup step doesn't happen automatically just because the plugin is installed — same gap `tinto-logistics`'s own daily scheduler had before its first run.

## Usage

Ask Claude things like:
- "Request a block of rooms from [hotel] for [season]" / "Hold rooms for [hotel]" — opens a new seasonal hold (Requested).
- "Did any hotels reply?" / "Check for hotel responses" — runs `check-hotel-replies` on demand instead of waiting for its scheduled firing.
- "The hotel got back to us — [weeks] are available, [week] isn't" — records the reply directly (Available / Not available), same as confirming one of `check-hotel-replies`'s own proposals.
- "[Tour] just sold, book the rooms" / "Convert the hotel hold for [tour]" — converts a confirmed block into a sold booking and bridges it into Room Blocks.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Rooming lists — `tinto-rooming-lists`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Booking-page publishing — `tinto-booking-pages`. Changing the cancellation-buffer policy or which lead-time rule applies to which touchpoint stays Peter/Nina's — this skill never edits those. Logging a hotel's payment obligation in `Supplier Payments` — that table is Finance's (Peter/Nina) territory and is never written by this skill, even once a block converts to Sold; see `finance-reference.md`'s money rule. A live, ongoing sync between `Room Blocks` and `Hotel Room Inventory` — the bridge is a one-time handoff at the moment of sale, and the two tables are allowed to diverge afterward. `check-hotel-replies` writing to Airtable on its own, without Tamara's confirmation — it only ever proposes; see that skill's escalation section for why.
