# Tinto Client Itinerary PDF

Building the dated, tour-specific day-by-day itinerary PDF that a selling client (winery, alumni association, etc.) uses to sell a tour and pass on to their own prospective guests — a standalone plugin, split out of `tinto-guest-communication` so it can be installed on its own.

## Overview

Distinct from the destination-level Pre/Post-Tour Planning Guide. It was originally bundled with Nélia's guest-communication plugin, but split out (2026-09-15) so it can be installed independently by anyone who needs it without also needing the rest of that plugin's daily guest-and-winery communications scope — Peter, for instance, who needs this but not everything in `tinto-guest-communication`.

## Components

| Skill | Purpose |
|---|---|
| `client-itinerary-pdf` | Walks through a short confirmation conversation (winery/client, then date/location/price/capacity/itinerary, one question at a time) before building anything. Web-searches branding for a brand-new winery and files it to Airtable rather than guessing, pulls the tour's linked Client record's logo/accent color for the PDF's branding (falling back to Tinto's own brand only if none is on file), and applies Tinto's standing no-em-dash copy cleanup rather than reproducing stored text verbatim. |

No agents or hooks — enforcement is already handled by Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, plus the pdf skill available in the session (bundled with Cowork by default).

## Usage

Ask Claude things like:
- "Create a new itinerary PDF for a tour we're trying to sell" / "...for [tour]" — walks through confirming the winery, dates, location, price, capacity, and itinerary one question at a time, then builds the tour-specific day-by-day guest document.

## Not in this plugin

Daily guest-and-winery email communications — `tinto-guest-communication`. Entering a new winery's Airtable record and branding — `tinto-add-winery-record` (this skill assumes that's already been done, or does it inline for a brand-new winery, but doesn't own that process). Both are separate installs in the same marketplace.
