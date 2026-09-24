# Tinto Client Itinerary PDF

Building the dated, tour-specific day-by-day itinerary PDF that a selling client (winery, alumni association, etc.) uses to sell a tour and pass on to their own prospective guests. A standalone plugin, split out of `tinto-guest-communication` so it can be installed on its own.

## Overview

Distinct from the destination-level Pre/Post-Tour Planning Guide. It was originally bundled with Nélia's guest-communication plugin, but split out (2026-09-15) so it can be installed independently by anyone who needs it without also needing the rest of that plugin's daily guest-and-winery communications scope. Peter, for instance, needs this but not everything in `tinto-guest-communication`.

**Delegates new-client handling to `tinto-add-winery-record` (2026-09-24).** When the conversational flow (below) hits a winery or client that isn't in Airtable yet, this skill no longer researches and files that record itself. It hands off to `tinto-add-winery-record`'s own skill, the same logo/brand-color web-search-and-confirm process, the same commission/discount/payment-terms boundary, so that logic lives in exactly one place instead of two copies that can drift apart. See Setup below for what this means for installation.

**Expands terse Airtable copy into real narrative voice, from real reference itineraries (2026-09-24).** `Itinerary Days` and `Tours.Overview` are shared with the live booking pages, so they're deliberately concise and can't be enriched at the source. This skill now expands that factual content into fuller narrative prose on the way into the PDF, using one real, previously-sent client itinerary per destination (`references/destination-itineraries/`) as the voice and structure model, rather than reproducing the terse Airtable copy verbatim. It never invents a specific personal or anecdotal detail that isn't confirmed, color comes only from the reference file itself (when a stop is confirmed still accurate) or from what Peter/Nélia say during the confirmation conversation. The full draft always goes back to Peter/Nélia for a check before being treated as final.

## Components

| Skill | Purpose |
|---|---|
| `client-itinerary-pdf` | Walks through a short confirmation conversation (winery/client, then date/location/price/capacity/itinerary, one question at a time) before building anything. For a brand-new winery, hands off to `tinto-add-winery-record` to source branding and file it to Airtable rather than duplicating that process. Pulls the tour's linked Client record's logo/accent color for the PDF's branding (falling back to Tinto's own brand only if none is on file), expands `Itinerary Days`/`Overview`'s terse, booking-page-shared copy into real narrative voice using a per-destination reference itinerary, and applies Tinto's standing no-em-dash copy cleanup. |

No agents or hooks. Enforcement is already handled by Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, plus the pdf skill available in the session (bundled with Cowork by default). **Also install `tinto-add-winery-record` alongside this plugin** if there's any chance a new client will come up mid-conversation. Without it, this skill can still handle every tour for an already-on-file client, but will stop and ask you to install that plugin (or add the record yourself) rather than improvising its own version of that research.

## Usage

Ask Claude things like:
- "Create a new itinerary PDF for a tour we're trying to sell" / "...for [tour]": walks through confirming the winery, dates, location, price, capacity, and itinerary one question at a time, then builds the tour-specific day-by-day guest document.

## Not in this plugin

Daily guest-and-winery email communications: `tinto-guest-communication`. Entering a new winery's Airtable record and branding: `tinto-add-winery-record`. This skill no longer does any of that research itself; it hands off to that plugin's skill when a new client comes up, and asks for it to be installed if it isn't already. Both are separate installs in the same marketplace.
