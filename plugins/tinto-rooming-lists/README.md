# Tinto Rooming Lists

Produces a per-tour rooming list — which guests are booked into which room, at which property — as a spreadsheet to send to a hotel or use internally.

## Overview

A reporting task built from confirmed `Reservations` (one row per booked room, not per traveler), not a booking-management or capacity-planning tool. The finished list is a plain, functional working document, not a branded guest-facing piece.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Status:** rebuilt 2026-09-15 twice, then extended 2026-09-16 for multi-hotel tours. First pass checked the original placeholder design against live production schema and a real tour's bookings and found several gaps (guest names, room numbers, blank Bed Configuration data). Second pass — the one that actually matters most — was checked against a **real Tinto rooming list** Sabrina supplied (`Rooming List_Linganore_08June.xlsx`), which turned out to have a specific, non-obvious structure this skill now matches exactly: a room-summary block (room number, names, headcount, bed configuration in Portuguese — "cama casal"/"2 camas") followed by per-guest passport rows (Full Name, Passport Number, Date of Birth, Expiration Date, pulled from `Participants.Passport Full Name` etc., distinct from the casual name), a fallback marker for passport data not yet in hand, and a TOTAL row. That marker reads "NO HOTEL" in the real file — Sabrina clarified this is Portuguese for "at the hotel," specific to a Portuguese-language tab, not a fixed universal string; the rebuilt skill translates it per tab (English tab: "AT HOTEL"; other languages: their own equivalent). The casual room-summary name field turned out to be exactly the Reservation's own `Buyer First Name` (the same field flagged in the first pass as unsuitable for passport purposes) — right for that column, wrong for the Full Name/passport columns, which now correctly pull `Participants.Passport Full Name` instead. Every rooming-list workbook is built with an English tab plus the hotel's own local language tab (determined from the matching Suppliers record's `Language Preference` field) unless that language is already English, in which case one tab is enough. `Food Restriction` is included by default per Sabrina (hotels typically provide breakfast). Not rebuilt around `Payment Status` — by the time this list is normally built (about a week before the tour), a still-unconfirmed reservation shouldn't exist, so that's an escalation case, not a filtering rule.

**Third pass, 2026-09-16:** live-testing the rebuilt skill against a real, currently-booked tour ("Tinto Presents the Joys of Southern Italy — Puglia," Sep 26–Oct 3 2027) surfaced a real design gap — that tour is a 3-city routing (Bari, Ostuni, Lecce), each with its own hotel, which the single-hotel design had no explicit handling for. The skill now builds **one workbook per hotel** for a multi-stop tour, using the `Tour Accommodation` table (linked from the Tour via `Accommodation Stops`) as the authoritative source of which hotels are on record — each stop row links directly to its own `Suppliers` record, which is now the primary path to that stop's language (a real link, not a text match), with the old Property-Name fuzzy match demoted to a fallback for single-stop tours with no `Tour Accommodation` row. Every stop's workbook is built from the same full reservation-based room/guest roster, on the assumption that the group keeps its room assignments for the whole tour — flagged in the skill as an assumption to escalate against, not a confirmed fact, since Tinto hasn't yet confirmed whether that ever isn't true. This is still a design-level rebuild verified against real schema and a real multi-stop tour's data, not yet a full end-to-end run confirmed with Tamara or against a live hotel send.

## Components

| Skill | Purpose |
|---|---|
| `rooming-lists` | Pulls booked-room data for a tour from `Reservations` (filtered directly by Tour) and their linked `Participants`/`Packages` records — room number (parsed from Notes), casual room names, headcount, bed configuration, per-guest passport details, food restrictions — and builds a spreadsheet (English tab, plus the hotel's local language tab when it isn't already English) via the xlsx skill, matching Tinto's real rooming-list format. For a tour with more than one hotel on record, builds one workbook per hotel using the `Tour Accommodation` table. Deliberately avoids the deprecated `Packages.Capacity` field for any "rooms remaining" question — points to the newer `Room Blocks` table instead. |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a rooming list is requested.

## Usage

Ask Claude things like:
- "Create a rooming list for [tour]" / "Who's in which room for [tour]?" — pulls the data and builds the spreadsheet.
- "Send the hotel our rooming list" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Room assignment changes or anything writing back to Airtable beyond the list itself are ordinary booking-data edits, not a special case of this skill.
