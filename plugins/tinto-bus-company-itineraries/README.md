# Tinto Bus Company Itineraries

Produces the plain, day-by-day pickup/dropoff schedule Tamara sends straight to a tour's Transport supplier — the document the driver actually works from.

## Overview

An operational document (times, places, pax counts, luggage handling, plain logistics language), not guest-facing marketing copy and not one of Tinto's branded PDFs. Built from every `Bookings (Confirmations)` row for the tour, in day/time order — not just rows tagged Transport, since the driver needs the whole day's sequence (winery visits, meals, comfort stops, the return to the hotel), not only formal transport bookings.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Rebuilt 2026-09-16** against a real reference document Sabrina provided (`BUS_2026_06_08_Linganore_alentejo.pdf`). The original placeholder assumed a branded PDF built from Supplier.Type = Transport rows alone; the real document is plain, unbranded text covering the whole day, with exact clock times and two on-the-ground contact names/numbers that don't come from Suppliers at all. Four fields were added to `Bookings (Confirmations)` (`Time`, `Stop Type`, `Location / Address Note`, `Logistics Note`) and one to `Tours` (`On-Tour Contacts`) to support this — see the skill's own "Rebuilt 2026-09-16" section for the full reasoning. **Historic tours won't have this data populated yet**; expect to escalate to Tamara for older tours until the new fields are filled in going forward.

## Components

| Skill | Purpose |
|---|---|
| `bus-company-itineraries` | Pulls every `Bookings (Confirmations)` row for a tour in day/time order, plus confirmed headcount and on-tour contacts from `Tours`, and composes the plain-text day-by-day driver schedule in European Portuguese, matching the reference document's exact layout and phrasing conventions. |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a tour's transport schedule is needed.

## Usage

Ask Claude things like:
- "Create the bus itinerary for [tour]" / "Put together the transport schedule for [tour]" — pulls the data and builds the document.
- "Send the driver our day-by-day pickup schedule" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Rooming lists — `tinto-rooming-lists`. Guest and winery communications, and the guest-facing client itinerary PDF — `tinto-guest-communication` / `tinto-client-itinerary-pdf` (those use a branded-PDF format this plugin deliberately does not; this document is a plain internal working document, matching what Tamara's transport suppliers actually receive today).
