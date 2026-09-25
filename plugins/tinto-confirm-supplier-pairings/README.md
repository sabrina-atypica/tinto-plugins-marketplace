# Tinto Confirm Supplier Pairings

Links a newly sold tour to its suppliers at the sales to ops handover, so the daily supplier emails (`tinto-logistics`) and the bus itinerary (`tinto-bus-company-itineraries`) know who is on the tour.

## Overview

Peter or Nélia sell a tour and create it with `tinto-add-new-tour`, which deliberately doesn't touch suppliers. From then on the tour shows up in Tamara's daily supplier summary as "sold, not yet linked to suppliers" until she runs this skill.

The skill copies the most recent tour at the same destination: its day-by-day stops go into `Bookings (Confirmations)` with dates shifted to the new start date, and each emailable supplier gets its touchpoint rows in `Supplier Booking Lead Times`. Tamara confirms once ("same itinerary and suppliers as <previous tour>?") and can swap individual suppliers before anything is written. Both tables are written in the same step, so they stay consistent.

If a supplier later turns out to be different for this tour, Tamara corrects it when the daily run is about to draft to that supplier. `daily-supplier-communications` then updates both tables and drafts to the right supplier.

**Rebuilt 2026-09-25 (0.3.0).** This plugin used to audit supplier pairings that had been guessed by matching a tour's region to a supplier's region. That audit is obsolete: on 2026-09-25 `Supplier Booking Lead Times` was synced to `Bookings (Confirmations)`, which had been verified against Nélia's 2027 ops workbooks. Ops spreadsheets are being retired, so Airtable is now the only record and new tours are linked here instead.

## Components

| Skill | Purpose |
|---|---|
| `link-tour-suppliers` | Finds Confirmed tours with no supplier links, picks the most recent tour at the same destination as the template, gets one confirmation from Tamara (with any swaps), then writes `Bookings (Confirmations)` and `Supplier Booking Lead Times` together and verifies the result. |

No agents or hooks. Every write is ordinary day-to-day Airtable data entry.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task of its own: the daily run in `tinto-logistics` lists the tours waiting to be linked, and Tamara runs this skill when she's ready.

## Usage

Ask Claude things like:
- "Link the suppliers for the new Linganore Alentejo tour"
- "Which tours still need suppliers linked?"
- "Do the handover for [tour]"

## Not in this plugin

Drafting or sending supplier emails, and correcting a supplier at drafting time: `tinto-logistics`. Hotel room holds and hotel touchpoint rows: `tinto-batch-book-hotel-rooms`. Creating the tour itself: `tinto-add-new-tour`. Guest-facing day-by-day copy (`Itinerary Days`): `tinto-client-itinerary-pdf`. Changing lead-time rules or the Airtable schema stays with Peter and Nina.
