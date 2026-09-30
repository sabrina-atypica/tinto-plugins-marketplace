# Tinto Guest Communication

Onboards Nélia (Tinto Travels' guest communication owner) onto the automated guest-and-winery communications process.

## Overview

This plugin covers Nélia's day-to-day operation of the guest journey: checking Airtable for what's due each day, drafting emails correctly, and reviewing/sending them. It deliberately does **not** cover supplier communications (Tamara's `tinto-logistics` plugin), booking-page publishing (`tinto-booking-pages`), client itinerary PDFs (`tinto-client-itinerary-pdf`), adding winery records (`tinto-add-winery-record`), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD (landing new clients — a separate, not-yet-built process). **Split 2026-09-15:** `add-winery-record` and `client-itinerary-pdf` used to be bundled in this plugin — they're now their own standalone plugins so someone who needs one of them (Peter, for instance) doesn't have to install the rest of Nélia's daily-communications scope to get it.

## Components

| Skill | Purpose |
|---|---|
| `daily-guest-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail (body from `Email Templates.Template HTML`, subject from `Email Templates.Subject` as of 2026-09-29), reviewing/sending, attachment handling, and the line between what Nélia can change herself and what stays with Peter/Nina. Since 0.10.0 it also reads the journey start date from Airtable's `Automation Config` (`GUEST_JOURNEY_START_DATE`) and never drafts an email dated before it, and never treats a booking with a missing Total Price as already paid. |
| `guest-inbox-triage` | The inbound half of the same daily check: reads new guest/winery replies, matches them to a Reservation or winery contact, classifies what each one needs, and proposes an Airtable update, a drafted reply, or a flag — always run together with `daily-guest-communications`, never on its own schedule. Also reads a winery's reply to an outstanding Winery Approval Request and proposes the Post-Tour Preference approval/decline update. |

No agents or hooks — enforcement of what Nélia can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors.

**The daily check needs a scheduled task to actually run automatically.** Installing this plugin alone doesn't make the daily check happen on its own — Sabrina (or whoever sets this up) needs to create a scheduled task on Nélia's account that invokes `daily-guest-communications` once a day; that skill's own instructions now say to also run `guest-inbox-triage` every time, so one scheduled task covers both. This is a one-time setup step outside the plugin itself.

**The Winery Approval Request touchpoint (End+1) is now part of the scheduled daily check** (2026-09-28) — it sets `Tours.Post-Tour Preference — Winery Approval` to Pending Winery Approval when it drafts, and `guest-inbox-triage` watches for the winery's reply to move it to Approved or Declined. No separate setup needed; both fields already exist on `Tours`.

**`guest-inbox-triage` needs `Client / Affiliation.Contact Email` populated to match winery replies.** This field was added 2026-09-28 and is likely blank for most wineries at first — winery-side matching only works once it's filled in per client; guest-side matching (via `Reservations.Buyer Email`) works immediately, no setup needed. `guest-inbox-triage` also applies a Gmail label (`Claude/Triaged`) to track what it's already reviewed — no manual Gmail setup needed, the skill creates the label itself the first time it runs if it doesn't already exist.

## Usage

Ask Claude things like:
- "Check today's guest emails" / "What's due today?" — runs the full daily check: scheduled outbound drafts (`daily-guest-communications`) and inbound triage of new replies (`guest-inbox-triage`) together.
- "Why didn't [guest] get an email?" / "Is this a duplicate?" — troubleshoots using `daily-guest-communications`.
- "Anything from guests today?" / "Any replies?" — same combined check, phrased from the inbound side.

## Not in this plugin

Supplier communications, rooming lists, and transport-company itineraries — Tamara's `tinto-logistics` plugin. Booking-page readiness auditing and publishing — `tinto-booking-pages`. Client itinerary PDFs — `tinto-client-itinerary-pdf`. Adding winery records — `tinto-add-winery-record`. All separate installs in the same marketplace.
