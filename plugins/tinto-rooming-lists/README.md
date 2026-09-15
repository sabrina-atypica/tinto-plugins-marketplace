# Tinto Rooming Lists

Produces a per-tour rooming list — which guests are booked into which room, at which property — as a spreadsheet to send to a hotel or use internally.

## Overview

A reporting task built from confirmed `Reservations` (one row per booked room, not per traveler), not a booking-management or capacity-planning tool. The finished list is a plain, functional working document, not a branded guest-facing piece.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Status:** this skill is still its original placeholder design — it has not yet been rebuilt or verified against real production data the way `daily-supplier-communications` and `daily-guest-communications` have been. Treat its output as a first draft worth double-checking until it's had that pass.

## Components

| Skill | Purpose |
|---|---|
| `rooming-lists` | Pulls booked-room data (guests, property, room type, occupancy, bed configuration) for a tour from `Reservations`/`Packages`, builds a plain spreadsheet via the xlsx skill. Deliberately avoids the deprecated `Packages.Capacity` field for any "rooms remaining" question — points to the newer `Room Blocks` table instead. |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a rooming list is requested.

## Usage

Ask Claude things like:
- "Create a rooming list for [tour]" / "Who's in which room for [tour]?" — pulls the data and builds the spreadsheet.
- "Send the hotel our rooming list" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Room assignment changes or anything writing back to Airtable beyond the list itself are ordinary booking-data edits, not a special case of this skill.
