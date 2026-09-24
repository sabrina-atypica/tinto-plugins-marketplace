# Tinto Add Winery Record

Entering an already-agreed client or affiliation's record and branding into Airtable, a standalone plugin split out of `tinto-guest-communication` so it can be installed on its own.

## Overview

This is ordinary data entry, the same write scope as any other record its user maintains, **not** a sales or business development action. Anyone on the team can use it (Nélia, Peter, or anyone else with Airtable access), not only Nélia. It was originally bundled with Nélia's guest-communication plugin, but split out (2026-09-15) so it can be installed independently by anyone who needs it without also needing the rest of that plugin's daily guest-and-winery communications scope, Peter, for instance, who needs this but not everything in `tinto-guest-communication`.

What gets entered here feeds the booking-page generator directly (Client Name, Logo, Brand Accent Color), so the skill validates against what that generator actually needs, not just what Airtable will accept, per the booking-page team's own contract for this table (2026-09-24).

## Components

| Skill | Purpose |
|---|---|
| `add-winery-record` | Filing an already-agreed client or affiliation's name, logo, and brand accent color into Airtable's `Client / Affiliation` table, validated against the booking-page generator's requirements: a clean guest-facing name, a transparent-background landscape logo (not just any image search result), and a valid hex accent color. Logo and color are sourced from the client's own website by web search, never from a chat upload (the Airtable connection can't push a chat-attached file into an attachment field). Also asks directly, when relevant, about a wine-club member discount (code, amount, sign-up URL), the one carve-out from the plugin's otherwise firm boundary against setting financial terms. Explicitly scoped to exclude Sales/Business Development (landing or negotiating a new client) and commission rate/payment terms, which stay Peter's call even though the write scope technically allows them. |

No agents or hooks, enforcement is already handled by Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, and web search.

## Usage

Ask Claude things like:
- "Add Barrel Oak Winery as a new client affiliation." Claude looks up their website and files the record with logo and brand color.
- "Add a new client to our table, it's a restaurant called [name]." Claude asks for anything missing, then files it the same way.

## Not in this plugin

Deciding whether to bring on a new client, negotiating commission terms, that's Sales/Business Development, a separate process. Daily guest-and-winery email communications and client itinerary PDFs, `tinto-guest-communication` and `tinto-client-itinerary-pdf`, separate installs in the same marketplace.
