# Tinto Logistics

Onboards Tamara (Tinto Travels' supplier operations owner) onto the automated supplier communications cycle, rooming lists, and transport-company itineraries.

## Overview

This plugin covers Tamara's day-to-day operation of the supplier side: checking Airtable for what reconfirmation touchpoints are due each day, drafting supplier emails (or flagging WhatsApp/Viber copy-over) correctly, building rooming lists, and producing day-by-day transport schedules for bus/driver companies. It deliberately does **not** cover guest or winery-client communications (Nélia's `tinto-guest-communication` plugin), system-wide automation rules (lead-time windows, the 70%-full trigger, schema — Peter/Nina's), or hotel batch-booking (a separate `batch-book-hotel-rooms` skill referenced but not bundled here).

## Components

| Skill | Purpose |
|---|---|
| `supplier-communications` | The core process: the three/four supplier reconfirmation touchpoints, channel handling (Email/WhatsApp/Other), per-supplier language, escalation, and what Tamara can change herself vs. what stays with Peter/Nina. |
| `rooming-lists` | Building a per-tour rooms-and-guests spreadsheet from confirmed Reservations — deliberately does not compute "rooms remaining" given a known data-accuracy issue on the Packages table's Capacity field. |
| `bus-company-itineraries` | Building a branded PDF day-by-day pickup/dropoff schedule for a tour's Transport supplier. |

No agents or hooks — enforcement of what Tamara can and can't change is already handled by her Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. `rooming-lists` also needs the xlsx skill available in the session; `bus-company-itineraries` needs the pdf skill. Both are bundled with Cowork by default.

## Usage

Ask Claude things like:
- "Check today's supplier emails" / "What needs reconfirming?" — runs the daily supplier check and drafting.
- "Create the rooming list for [tour]" — builds the rooms-and-guests spreadsheet.
- "Put together the bus itinerary for [tour]" — builds the transport company's day-by-day schedule.

## Not in this plugin

Guest and winery-client communications, guest itinerary PDFs, and booking-page publishing — Nélia's equivalent `tinto-guest-communication` plugin, a separate install in the same marketplace. Hotel batch-booking (`batch-book-hotel-rooms`) is referenced in `supplier-communications` but is its own skill, not bundled here yet.
