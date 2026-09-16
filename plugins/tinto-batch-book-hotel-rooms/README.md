# Tinto Batch Book Hotel Rooms

Requests a new seasonal hotel room hold, or converts a held block into a sold tour's actual booking.

## Overview

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses — a block of rooms is requested from a hotel ahead of any specific tour sale, for a whole upcoming season at once, then converted to a real booking once a tour sells against it. This plugin covers both halves of that process. Requesting a hold writes to the production base's `Hotel Room Inventory` table, including the `Cancellation Deadline` computed from the hotel's own confirmed cancellation policy. Converting a sold block updates that same row to Sold and creates or updates three `Supplier Booking Lead Times` rows for the Tour × Hotel pairing (Rooming List, Room Count Reconciliation, and Hotel Final Confirmation) — the step that actually plugs the sale into the rest of the automation.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who manages hotel blocks (typically Tamara, sometimes Nélia) — not gated behind Tamara's restricted daily-cycle plugin.

**Reviewed and rebuilt against live production schema and data, 2026-09-16** (see below). **Updated 2026-09-16 (v0.3.0):** both halves now write a human-readable `Notes` narrative on the `Hotel Room Inventory` row (append-don't-overwrite, matching the base's `[DRAFTED ...]`/`[SENT ...]` marker convention), and the scope boundary below now says explicitly that this skill never touches `Supplier Payments`.

## Components

| Skill | Purpose |
|---|---|
| `batch-book-hotel-rooms` | Half 1: requesting a new room hold from a hotel (drafts the request, records `Hotel Room Inventory` as Available with room type, requested date, and a policy-derived cancellation deadline). Half 2: converting a sold block (updates `Hotel Room Inventory` to Sold, then creates or updates the pairing's three `Supplier Booking Lead Times` rows — Rooming List, Room Count Reconciliation, Hotel Final Confirmation — most of which already exist as region-matched drafts from `confirm-supplier-pairings` by the time a tour sells). |

No agents or hooks — the skill's own escalation guidance covers the judgment calls (no confirmed cancellation policy, a genuine duplicate lead-time row versus an expected pre-seeded draft, anything that edges toward Sales/BD negotiation).

**Note on Final Reconfirmation:** the 28-day, calendar-independent "Final Reconfirmation" touchpoint documented in `logistics-reference.md` for Hotel/Winery/Restaurant/Transport currently has no rows for Hotel at all in production (only Winery/Restaurant/Transport got them in the 2026-09-09 bulk wire) — a known, unresolved data gap outside this skill's scope, not something this skill invents or silently patches.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. No scheduled task needed — this is run on demand, whenever a hold is requested or a block sells.

## Usage

Ask Claude things like:
- "Request a block of rooms from [hotel] for [season]" / "Hold rooms for [hotel]" — opens a new seasonal hold.
- "[Tour] just sold, book the rooms" / "Convert the hotel hold for [tour]" — converts a held block into a sold booking.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Rooming lists — `tinto-rooming-lists`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Booking-page publishing — `tinto-booking-pages`. Changing the cancellation-buffer policy or which lead-time rule applies to which touchpoint stays Peter/Nina's — this skill never edits those. Logging a hotel's payment obligation in `Supplier Payments` — that table is Finance's (Peter/Nina) territory and is never written by this skill, even once a block converts to Sold; see `finance-reference.md`'s money rule.
