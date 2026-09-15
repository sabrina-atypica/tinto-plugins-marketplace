---
name: bus-company-itineraries
description: >
  This skill should be used when Tamara asks to "create the bus itinerary
  for [tour]," "put together the transport schedule for [tour]," "send the
  driver our day-by-day pickup schedule," or needs a day-by-day
  pickup/dropoff document to send to a tour's Transport supplier (a bus or
  driver company).
metadata:
  version: "0.1.0"
---

# Bus Company Itineraries

Produce a day-by-day pickup/dropoff schedule for a tour's Transport supplier, as a branded PDF Tamara can send directly. This is an **operational document for the transport company**, not guest-facing marketing copy — write it in plain, exact, logistics language (times, addresses, pax counts, pickup points), not the narrative tone used in guest materials.

## Where the data comes from

**Primary source of truth: the production Airtable base's `Bookings (Confirmations)` table**, filtered to the requested tour and to rows whose linked `Supplier` has `Type` = Transport. Each row is one supplier slot for one day of the tour, carrying `Day #`, `Time Slot` (Morning/Lunch/Afternoon/Evening), and an operational `Activity Description`. Pull every Transport row for the tour, in day/time-slot order — don't assume a fixed pattern (e.g. "pickup every morning, dropoff every evening"), since a tour's actual bus schedule can include mid-day transfers, a day trip requiring transport, or a day with none at all.

If `Bookings (Confirmations)` looks sparse or seems to be missing a day a bus is obviously needed (e.g. arrival/departure transfers), check `Standard Itineraries` for that destination as a baseline reference, but **do not treat it as confirmed data** — every row there is still marked `Confidence: Draft - needs confirming` as of 2026-09-10, and several destinations were reconstructed rather than pulled from real bookings. Flag any gap to Tamara rather than filling it in from the draft baseline silently.

Also pull, for header/context use:

- The final confirmed headcount for the tour (for pax counts per pickup) — check `Reservations` linked to the tour, or ask Tamara if final numbers aren't locked yet, rather than guessing.
- The transport supplier's own contact details and `Language Preference` (from `Suppliers`) — the document should be in the supplier's preferred language per the `supplier-communications` skill's language rule, not automatically English.

## Document structure

One page (or continuous flow) per tour, ordered by day:

- Header: tour name, dates, transport supplier name, total pax.
- Per day: date, day number, each pickup/dropoff/transfer entry with time, location name and address (as specific as the source data gives — flag if an address is missing rather than inventing one), and any note relevant to the driver (luggage count, an early/late flag, a stop that isn't obvious from the location name alone).

## Branding

Build the PDF using the pdf skill (read its SKILL.md before generating), styled with Tinto Travels' existing document brand — see `references/tinto-brand.md` for the palette, typefaces, and layout conventions already established for Tinto's other branded PDFs (the Pre/Post-Tour Planning Guide). Keep the styling restrained here: this is a working document for a supplier, not a guest-facing piece, so brand consistency matters more than visual richness — a clean header/footer treatment and correct fonts/colors is enough, no need to replicate every layout flourish.

## Escalate rather than guess

If a day is missing a Transport row entirely, an address or time looks incomplete, or the tour's final headcount isn't confirmed yet, say so plainly and ask Tamara rather than inventing a plausible-looking schedule — a driver working from a wrong or incomplete schedule is a real operational failure, not just an inconvenience.
