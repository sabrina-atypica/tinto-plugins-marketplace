# Tinto Bus Company Itineraries

Produces the plain, day-by-day pickup/dropoff schedule Tamara sends straight to a tour's Transport supplier — the document the driver actually works from.

## Overview

An operational document (times, places, pax counts, luggage handling, plain logistics language), not guest-facing marketing copy and not one of Tinto's branded PDFs. Built from every `Bookings (Confirmations)` row for the tour, in day/time order — not just rows tagged Transport, since the driver needs the whole day's sequence (winery visits, meals, comfort stops, the return to the hotel), not only formal transport bookings.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who needs it — not gated behind Tamara's restricted daily-cycle plugin.

**Rebuilt 2026-09-16** against a real reference document Sabrina provided (`BUS_2026_06_08_Linganore_alentejo.pdf`). The original placeholder assumed a branded PDF built from Supplier.Type = Transport rows alone; the real document is plain, unbranded text covering the whole day, with exact clock times and two on-the-ground contact names/numbers that don't come from Suppliers at all. Four fields were added to `Bookings (Confirmations)` (`Time`, `Stop Type`, `Location / Address Note`, `Logistics Note`) and one to `Tours` (`On-Tour Contacts`) to support this — see the skill's own "Rebuilt 2026-09-16" section for the full reasoning. **Historic tours won't have this data populated yet**; expect to escalate to Tamara for older tours until the new fields are filled in going forward.

**Extended 2026-09-17:** five more real examples from Tamara (Alentejo, Loire Valley, Northern Adriatic & Slovenia, Southern Tuscany & Umbria, Castille & León) showed the format isn't one universal template — language, header shape, and level of detail all vary by destination. `skills/bus-company-itineraries/references/destination-formats.md` now documents each destination's own convention; the skill looks up the tour's `Location` and follows that destination's format rather than defaulting to Alentejo's. None of those five examples correspond to a tour currently in production Airtable (all 2026-dated; production only has 2027 tours) — they're format references only.

**Onboarding check-in added 2026-09-24:** the first time this skill runs for Tamara after handover, it leads with a short coverage summary before doing anything else — which destinations already have a template on file (Alentejo, Loire Valley, Northern Adriatic & Slovenia, Southern Tuscany & Umbria, Castille & León), and which don't yet, split by urgency: Porto & Douro, Peloponnese (Greece), Puglia, and Austria already have real 2027 tours running with no template on file, while Coastal Tuscany and Vinho Verde have neither a template nor a tour yet. It asks whether she has a template ready for one of the missing ones (the urgent ones especially) to drop in now, and if not, waits and asks again only when a real request for that destination comes up, rather than asking her to produce templates for every gap up front. (`Tours.Location` also has a stray unused "Douro" choice, distinct from "Porto & Douro" — no real tour uses it, so it isn't treated as a destination of its own.)

## Components

| Skill | Purpose |
|---|---|
| `bus-company-itineraries` | Pulls every `Bookings (Confirmations)` row for a tour in day/time order, plus confirmed headcount and on-tour contacts from `Tours`, and composes the plain-text day-by-day driver schedule in that destination's own language and layout convention (see `references/destination-formats.md`). |

No agents or hooks.

## Setup

Requires the org's existing **Airtable** connector. No scheduled task needed — this is run on demand, whenever a tour's transport schedule is needed.

## Usage

Ask Claude things like:
- "Create the bus itinerary for [tour]" / "Put together the transport schedule for [tour]" — pulls the data and builds the document.
- "Send the driver our day-by-day pickup schedule" — same skill, framed as an external send.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin. Confirming supplier pairings — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Rooming lists — `tinto-rooming-lists`. Guest and winery communications, and the guest-facing client itinerary PDF — `tinto-guest-communication` / `tinto-client-itinerary-pdf` (those use a branded-PDF format this plugin deliberately does not; this document is a plain internal working document, matching what Tamara's transport suppliers actually receive today).
