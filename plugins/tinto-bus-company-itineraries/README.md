# Tinto Bus Company Itineraries

Produces a day-by-day pickup/dropoff schedule for a tour's Transport supplier, as a branded PDF.

## Overview

An operational document for the transport company (times, addresses, pax counts, pickup points, plain logistics language), not guest-facing marketing copy. Built from `Bookings (Confirmations)` Transport rows for the tour; `Standard Itineraries` is checked only as an unconfirmed baseline reference when real data looks sparse, never treated as fact on its own.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Status:** this skill is still its original placeholder design — it has not yet been rebuilt or verified against real production data the way `daily-supplier-communications` and `daily-guest-communications` have been. Treat its output as a first draft worth double-checking until it's had that pass.

## Components

| Skill | Purpose |
|---|---|
| `bus-company-itineraries` | Pulls Transport rows for a tour from `Bookings (Confirmations)` in day/time-slot order, plus confirmed headcount and the supplier's language preference, and builds a branded PDF via the pdf skill using `references/tinto-brand.md` (kept in sync manually with the copy in `tinto-client-itinerary-pdf`). |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a tour's transport schedule is needed.

## Usage

Ask Claude things like:
- "Create the bus itinerary for [tour]" / "Put together the transport schedule for [tour]" — pulls the data and builds the PDF.
- "Send the driver our day-by-day pickup schedule" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Rooming lists — `tinto-rooming-lists`. Guest and winery communications, and the guest-facing client itinerary PDF — `tinto-guest-communication` / `tinto-client-itinerary-pdf` (a shared brand reference file is duplicated between the two plugins, not linked, since each is independently installable).
