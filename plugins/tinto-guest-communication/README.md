# Tinto Guest Communication

Onboards Nélia (Tinto Travels' guest communication owner) onto the automated guest-and-winery communications process, client itinerary documents, and booking-page publishing.

## Overview

This plugin covers Nélia's day-to-day operation of the guest journey: checking Airtable for what's due each day, drafting emails correctly, reviewing/sending them, building client itinerary PDFs, auditing and publishing tour booking pages, and adding winery records. It deliberately does **not** cover supplier communications (Tamara's `tinto-logistics` plugin), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD (landing new clients — a separate, not-yet-built process).

**Booking-page publishing has one real precondition — read `booking-page-publishing`'s SKILL.md before using it.** As of 2026-09-14, a real guest payment on a live booking page may not be reaching Airtable at all (a broken webhook step) — confirm this is fixed with Sabrina, Peter, or Nina before publishing any page for the first time. Publishing itself needs no repo or GitHub access as of this version — it's a single authenticated HTTPS call to the production Worker, using only Airtable access and the standard Cowork shell.

## Components

| Skill | Purpose |
|---|---|
| `daily-guest-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail, reviewing/sending, attachment handling, and the line between what Nélia can change herself and what stays with Peter/Nina. |
| `add-winery-record` | Entering an already-agreed winery client's record and branding into Airtable. |
| `client-itinerary-pdf` | Building the dated, tour-specific day-by-day itinerary PDF for a tour that's about to sell, being pitched, or already sold — the document the selling client uses to sell the tour and pass on to their own guests, distinct from the destination-level Pre/Post-Tour Planning Guide. Walks through a short confirmation conversation (winery/client, then date/location/price/capacity/itinerary) before building anything, and web-searches branding for a brand-new winery rather than guessing. |
| `booking-page-publishing` | Auditing a tour's Airtable data against booking-page requirements and publishing it live via the `/api/publish-tour` endpoint (Airtable + a shell `curl` call — no GitHub needed), with an explicit warning about the current guest-payment webhook issue. |

No agents or hooks — enforcement of what Nélia can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors. The `client-itinerary-pdf` skill also needs the pdf skill available in the session (bundled with Cowork by default). `booking-page-publishing` needs only Airtable access and the standard Cowork shell (for one `curl` call) — no device bridge, repo access, or GitHub required.

**The daily check needs a scheduled task to actually run automatically.** Installing this plugin alone doesn't make the daily check happen on its own — Sabrina (or whoever sets this up) needs to create a scheduled task on Nélia's account that invokes the `daily-guest-communications` skill once a day. This is a one-time setup step outside the plugin itself.

## Usage

Ask Claude things like:
- "Check today's guest emails" / "What's due today?" — runs the daily check and drafting.
- "Why didn't [guest] get an email?" / "Is this a duplicate?" — troubleshoots using the same skill.
- "Add [winery] as a new client, here's their logo" — files a winery record and branding.
- "Create a new itinerary PDF for a tour we're trying to sell" / "...for [tour]" — walks through confirming the winery, dates, location, price, capacity, and itinerary one question at a time, then builds the tour-specific day-by-day guest document.
- "Is [tour]'s booking page ready? Publish it" — audits and, where possible, publishes.

## Not in this plugin

Supplier communications, rooming lists, and transport-company itineraries — Tamara's equivalent `tinto-logistics` plugin, a separate install in the same marketplace.
