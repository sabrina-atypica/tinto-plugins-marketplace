# Tinto Confirm Supplier Pairings

Audits region-matched tour-supplier pairings against real itinerary evidence, destination by destination, so `daily-supplier-communications` (in `tinto-logistics`) can draft safely.

## Overview

Most of production's `Supplier Booking Lead Times` table was built by matching a tour's region against a supplier's region — "this supplier operates in the same area as this tour," not "this specific supplier is confirmed for this specific tour." This skill walks Tamara through the region-matched pairings destination by destination, checks each one against real itinerary evidence (`Bookings (Confirmations)`) and Gmail history, and writes the result directly onto the rows `daily-supplier-communications` reads.

**Split 2026-09-15:** this used to be bundled inside `tinto-logistics`. It's now its own standalone plugin, installable by anyone on the team who runs it — not gated behind Tamara's restricted daily-cycle plugin. It should still be run once, destination by destination, before `tinto-logistics`'s daily scheduler runs for the first time — `daily-supplier-communications` will not draft to a pairing that hasn't been through this review.

## Components

| Skill | Purpose |
|---|---|
| `confirm-supplier-pairings` | Three phases per destination batch: confirm **who** the supplier actually is (against itinerary evidence), backfill **what's already happened** with them for this tour (against Gmail), and flag (never change) **whether the timing looks right**. Resumable across sessions and destinations — only ever pulls rows not yet reviewed. |

No agents or hooks — every write this skill makes is an append to `Notes` or a `Supplier` link correction, both well within ordinary day-to-day Airtable access.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors.

No scheduled task needed — this is a manually-run audit, not a daily automated process. Run it destination by destination, in whatever order suits (soonest tour start dates first is a reasonable default), and pick it back up later exactly where it left off.

## Usage

Ask Claude things like:
- "Audit the supplier pairings for [destination]" / "Confirm suppliers for [destination]" — runs a full destination batch through all three phases.
- "Is [supplier] really booked for [tour]?" / "What's already been sent to [supplier]?" — spot-checks a single pairing using the same evidence and logic.

## Not in this plugin

The daily supplier-check-and-draft cycle itself — Tamara's restricted `tinto-logistics` plugin (won't draft to a pairing this skill hasn't cleared). Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Rooming lists — `tinto-rooming-lists`. Bus company itineraries — `tinto-bus-company-itineraries`. Guest and winery communications — `tinto-guest-communication`. Booking-page publishing — `tinto-booking-pages`. Changing lead-time rules, trigger timing, or Airtable schema itself stays Peter/Nina's — this skill only ever flags a timing concern, never edits one.
