# Tinto Add Supplier

Entering an already-agreed supplier's record into Airtable's `Suppliers` table, a standalone plugin mirroring `tinto-add-winery-record`'s architecture for the client-facing side.

## Overview

This is ordinary data entry, the same write scope as any other record its user maintains, **not** a sales or business development action. Anyone on the team can use it (Tamara, Nélia, Peter, or anyone else with Airtable access), not only Tamara or Nélia.

Every other skill in this marketplace that touches `Suppliers` (`tinto-add-new-tour`, `link-tour-suppliers`, `tinto-batch-book-hotel-rooms`, `daily-supplier-communications`) explicitly refuses to invent a record for a supplier that doesn't exist yet, it just says so. This plugin is what actually creates one.

The most costly gap this closes: `Hotel Cancellation Policy` and `Cancellation Policy Confirmed Date` are real, load-bearing data, `tinto-batch-book-hotel-rooms` reads them to compute a confirmed hold's `Cancellation Deadline`, and Room Count Reconciliation's lead time is meant to run against the same figure, but as of 2026-09-28 only about 5 of 34 hotels had it filled in. This plugin never web-sources that specific field, it only gets written from a real answer someone already has confirmed with the hotel directly; a wrong figure here costs real money, so the design deliberately doesn't try to shortcut it the way it shortcuts lower-stakes fields like property photos.

## Components

| Skill | Purpose |
|---|---|
| `add-supplier` | Filing an already-agreed supplier's name, type, region, and operational contact details into Airtable's `Suppliers` table, deduped against existing records first. Actively interviews for the fields real operation depends on (email, preferred communication channel, language preference, payment terms); accepts Contact Name, Phone, and Comments opportunistically only, since a base-wide audit found no automated consumer for any of the three. For hotels, web-sources property photos and a guest description from the hotel's own site (confirmed before writing), but never the Hotel Cancellation Policy, that field is asked of Tamara/Nélia/Peter directly or left blank pending follow-up, never guessed or pulled from a public webpage that may not reflect Tinto's actual negotiated terms. |

No agents or hooks, enforcement is already handled by Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, and web search (for hotel photos/description only).

## Usage

Ask Claude things like:
- "Add a new hotel supplier, Herdade da Malhadinha Nova, in the Alentejo." Claude looks up the region, interviews for contact details, and asks whether the cancellation policy is already confirmed before touching that field.
- "We're working with a new tour guide in Porto now, here's her contact info." Claude files the record from what's given, asking only for what's still missing.

## Not in this plugin

Deciding whether to bring on a new supplier or negotiating rates/terms, that's Sales/Business Development, a separate process. `Supplier Payments` (Finance's territory) and linking a supplier to a specific tour (`link-tour-suppliers` or `tinto-batch-book-hotel-rooms`, separate installs in the same marketplace).
