# Tinto Rooming Lists

Produces a per-tour rooming list — which guests are booked into which room, at which property — as a spreadsheet to send to a hotel or use internally.

## Overview

A reporting task built from confirmed `Reservations` (one row per booked room, not per traveler), not a booking-management or capacity-planning tool. The finished list is a plain, functional working document, not a branded guest-facing piece.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Status:** rebuilt 2026-09-15 against live production schema and a real tour's bookings, after the original placeholder design was checked and found to have several real gaps. What changed: guest names now pull from linked `Participants` records (clean, separate First/Last Name) instead of the Reservation's own `Buyer First Name` field, which in practice is a messy human-typed string combining two people, nicknames, and asides ("Jennifer & Michael Howse," "Dave & Kathy Muenz (Complimentary)"); room numbers are now extracted from the existing "Room 1.", "Room 2." convention already present in Reservations' own `Notes` field on real bookings, since no structured room-number field exists anywhere in Airtable; `Bed Configuration` blanks (confirmed on at least the Alentejo tours' packages) are now shown as blank rather than silently guessed; `Room Request Tags`/`Room Request Notes` on Participants are now pulled in when populated; and each guest's `Food Restriction` is now included by default, since the hotels these lists go to typically provide breakfast and need the dietary/allergy information. Not rebuilt around `Payment Status` — by the time this list is normally built (about a week before the tour), a still-unconfirmed reservation shouldn't exist, so that's an escalation case, not a filtering rule. This is a design-level rebuild verified against live schema and real records, not yet a full end-to-end run confirmed with Tamara or against a live hotel send.

## Components

| Skill | Purpose |
|---|---|
| `rooming-lists` | Pulls booked-room data for a tour from `Reservations` (filtered directly by Tour) and their linked `Participants`/`Packages` records — guest names, property, room type, occupancy, bed configuration, room number (parsed from Notes when present), food restrictions, and any stated room-request notes — then builds a plain spreadsheet via the xlsx skill. Deliberately avoids the deprecated `Packages.Capacity` field for any "rooms remaining" question — points to the newer `Room Blocks` table instead. |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a rooming list is requested.

## Usage

Ask Claude things like:
- "Create a rooming list for [tour]" / "Who's in which room for [tour]?" — pulls the data and builds the spreadsheet.
- "Send the hotel our rooming list" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Room assignment changes or anything writing back to Airtable beyond the list itself are ordinary booking-data edits, not a special case of this skill.
