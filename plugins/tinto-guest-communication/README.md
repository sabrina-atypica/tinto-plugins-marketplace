# Tinto Guest Communication

Onboards Nélia (Tinto Travels' guest communication owner) onto the automated guest-and-winery communications process.

## Overview

This plugin covers Nélia's day-to-day operation of the guest journey: checking Airtable for what's due each day, drafting emails correctly, and reviewing/sending them. It deliberately does **not** cover supplier communications (Tamara's `tinto-logistics` plugin), booking-page publishing (`tinto-booking-pages`), client itinerary PDFs (`tinto-client-itinerary-pdf`), adding winery records (`tinto-add-winery-record`), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD (landing new clients — a separate, not-yet-built process). **Split 2026-09-15:** `add-winery-record` and `client-itinerary-pdf` used to be bundled in this plugin — they're now their own standalone plugins so someone who needs one of them (Peter, for instance) doesn't have to install the rest of Nélia's daily-communications scope to get it.

## Components

| Skill | Purpose |
|---|---|
| `daily-guest-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail, reviewing/sending, attachment handling, and the line between what Nélia can change herself and what stays with Peter/Nina. |

No agents or hooks — enforcement of what Nélia can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors.

**The daily check needs a scheduled task to actually run automatically.** Installing this plugin alone doesn't make the daily check happen on its own — Sabrina (or whoever sets this up) needs to create a scheduled task on Nélia's account that invokes the `daily-guest-communications` skill once a day. This is a one-time setup step outside the plugin itself.

## Usage

Ask Claude things like:
- "Check today's guest emails" / "What's due today?" — runs the daily check and drafting.
- "Why didn't [guest] get an email?" / "Is this a duplicate?" — troubleshoots using the same skill.

## Not in this plugin

Supplier communications, rooming lists, and transport-company itineraries — Tamara's `tinto-logistics` plugin. Booking-page readiness auditing and publishing — `tinto-booking-pages`. Client itinerary PDFs — `tinto-client-itinerary-pdf`. Adding winery records — `tinto-add-winery-record`. All separate installs in the same marketplace.
