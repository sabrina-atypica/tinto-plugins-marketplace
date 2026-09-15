# Tinto Batch Book Hotel Rooms

Requests a new seasonal hotel room hold, or converts a held block into a sold tour's actual booking.

## Overview

Hotels are the one supplier category that doesn't follow the rolling, per-tour first-contact model every other category uses — a block of rooms is requested from a hotel ahead of any specific tour sale, for a whole upcoming season at once, then converted to a real booking once a tour sells against it. This plugin covers both halves of that process, both of which write to the production base's `Hotel Room Inventory` table (and, on conversion, a new `Supplier Booking Lead Times` row — the step that actually plugs the sale into the rest of the automation).

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who manages hotel blocks (typically Tamara, sometimes Nélia) — not gated behind Tamara's restricted daily-cycle plugin.

## Components

| Skill | Purpose |
|---|---|
| `batch-book-hotel-rooms` | Half 1: requesting a new room hold from a hotel (drafts the request, records `Hotel Room Inventory` as Available). Half 2: converting a sold block (updates `Hotel Room Inventory` to Sold, creates the `Supplier Booking Lead Times` row that makes Room Count Reconciliation, Hotel Final Confirmation, and Final Reconfirmation actually fire). |

No agents or hooks — the skill's own escalation guidance covers the judgment calls (no confirmed cancellation policy, possible duplicate lead-time row, anything that edges toward Sales/BD negotiation).

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. No scheduled task needed — this is run on demand, whenever a hold is requested or a block sells.

## Usage

Ask Claude things like:
- "Request a block of rooms from [hotel] for [season]" / "Hold rooms for [hotel]" — opens a new seasonal hold.
- "[Tour] just sold, book the rooms" / "Convert the hotel hold for [tour]" — converts a held block into a sold booking.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Rooming lists — `tinto-rooming-lists`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Booking-page publishing — `tinto-booking-pages`. Changing the cancellation-buffer policy or which lead-time rule applies to which touchpoint stays Peter/Nina's — this skill never edits those.
