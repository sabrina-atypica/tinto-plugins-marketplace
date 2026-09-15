# Tinto Add Winery Record

Entering an already-agreed winery client's record and branding into Airtable — a standalone plugin, split out of `tinto-guest-communication` so it can be installed on its own.

## Overview

This is ordinary data entry — the same write scope as any other record its user maintains — **not** a sales or business-development action. It was originally bundled with Nélia's guest-communication plugin, but split out (2026-09-15) so it can be installed independently by anyone who needs it without also needing the rest of that plugin's daily guest-and-winery communications scope — Peter, for instance, who needs this but not everything in `tinto-guest-communication`.

## Components

| Skill | Purpose |
|---|---|
| `add-winery-record` | Filing an already-agreed winery client's name, logo, and brand colors into Airtable. Explicitly scoped to exclude Sales/BD (landing or negotiating a new client) and commission/discount-code/payment-terms fields, which stay Peter's call even though the write scope technically allows them. |

No agents or hooks — enforcement is already handled by Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector.

## Usage

Ask Claude things like:
- "Add [winery] as a new client, here's their logo" — files a winery record and branding.

## Not in this plugin

Deciding whether to bring on a new client, negotiating commission terms — that's Sales/BD, a separate process. Daily guest-and-winery email communications and client itinerary PDFs — `tinto-guest-communication` and `tinto-client-itinerary-pdf`, separate installs in the same marketplace.
