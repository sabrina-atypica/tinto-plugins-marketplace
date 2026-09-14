# Tinto Guest Communication

Onboards Nélia (Tinto Travels' guest communication owner) onto the automated guest-and-winery communications process and client itinerary documents.

## Overview

This plugin covers Nélia's day-to-day operation of the guest journey: checking Airtable for what's due each day, drafting emails correctly, reviewing/sending them, building client itinerary PDFs, and adding winery records. It deliberately does **not** cover supplier communications (Tamara's `tinto-logistics` plugin), booking-page publishing (its own separate `tinto-booking-pages` plugin — not currently part of Nélia's install), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD (landing new clients — a separate, not-yet-built process).

## Components

| Skill | Purpose |
|---|---|
| `daily-guest-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail, reviewing/sending, attachment handling, and the line between what Nélia can change herself and what stays with Peter/Nina. |
| `add-winery-record` | Entering an already-agreed winery client's record and branding into Airtable. |
| `client-itinerary-pdf` | Building the dated, tour-specific day-by-day itinerary PDF for a tour that's about to sell, being pitched, or already sold — the document the selling client uses to sell the tour and pass on to their own guests, distinct from the destination-level Pre/Post-Tour Planning Guide. Walks through a short confirmation conversation (winery/client, then date/location/price/capacity/itinerary) before building anything, and web-searches branding for a brand-new winery rather than guessing. |

No agents or hooks — enforcement of what Nélia can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. The `client-itinerary-pdf` skill also needs the pdf skill available in the session (bundled with Cowork by default).

**The daily check needs a scheduled task to actually run automatically.** Installing this plugin alone doesn't make the daily check happen on its own — Sabrina (or whoever sets this up) needs to create a scheduled task on Nélia's account that invokes the `daily-guest-communications` skill once a day. This is a one-time setup step outside the plugin itself.

## Usage

Ask Claude things like:
- "Check today's guest emails" / "What's due today?" — runs the daily check and drafting.
- "Why didn't [guest] get an email?" / "Is this a duplicate?" — troubleshoots using the same skill.
- "Add [winery] as a new client, here's their logo" — files a winery record and branding.
- "Create a new itinerary PDF for a tour we're trying to sell" / "...for [tour]" — walks through confirming the winery, dates, location, price, capacity, and itinerary one question at a time, then builds the tour-specific day-by-day guest document.

## Not in this plugin

Supplier communications, rooming lists, and transport-company itineraries — Tamara's `tinto-logistics` plugin. Booking-page readiness auditing and publishing — its own separate `tinto-booking-pages` plugin (split out 2026-09-14; not currently sent to Nélia as an update to this one). Both are separate installs in the same marketplace.
