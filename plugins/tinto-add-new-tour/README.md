# Tinto Add New Tour

Creating a brand-new `Tours` record in Airtable when Tinto has a real, confirmed tour that doesn't exist there yet, a standalone plugin in the same marketplace as the rest of Tinto's Airtable data-entry skills.

## Overview

Every tour currently in Airtable arrived there through a one-time spreadsheet migration in September 2026, not through an ongoing intake process. This plugin is that ongoing process: whenever Peter, Nélia, or anyone else has a new tour to record (a new departure for an existing client, a brand-new client's first tour, or a new destination), this is what creates the `Tours` record properly, with the fields other skills actually depend on, rather than leaving it half-filled or guessed.

**Deliberately scoped to the tour's own identity, not everything a tour eventually needs.** It creates Tour Name, Location, the linked Client, Start/End Date, Status, Price per Person, Max Participants, Pickup/Drop-off Location, Overview, What's Included/Not Included, Tour Highlights, Hero Facts, Notes, and a generated Slug. It also converts the tour's `Room Block` (already created by `tinto-batch-book-hotel-rooms` at the point of sale) into real, priced `Packages` rows, confirming Double and Solo pricing against the same destination's most recent tour rather than the hotel's own rate quote, since Peter sets the guest-facing price himself. It deliberately does not create the day-by-day `Itinerary Days` (that's `tinto-client-itinerary-pdf`'s job, which already has a confirm-and-draft flow for it), and it never touches hotel-side room holding or capacity (`Room Blocks`, `Hotel Room Inventory`, `tinto-batch-book-hotel-rooms`'s job entirely) or taking a tour live for online booking (`tinto-booking-pages`'s job, a separate later decision).

## Components

| Skill | Purpose |
|---|---|
| `add-new-tour` | Walks through confirming a new tour's client, destination, dates, status, price, capacity, and cover-page content one question at a time, then creates the `Tours` record. Hands off to `tinto-add-winery-record` for a brand-new client rather than duplicating that research, converts the tour's already-existing `Room Block` into priced `Packages` rows (confirming Double/Solo pricing with Peter against the same destination's most recent tour), and hands off (when ready to build day-by-day content) to `tinto-client-itinerary-pdf` rather than duplicating its itinerary-confirmation flow. |

No agents or hooks, enforcement is already handled by Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector. **Also install `tinto-add-winery-record`** alongside this plugin if there's any chance a brand-new client will come up (without it, this skill can still handle a new tour for an already-on-file client, but will stop and ask you to install that plugin or add the client record yourself first).

## Usage

Ask Claude things like:
- "Add a new tour: [client], [destination], [dates]." Claude confirms the client, asks through the remaining details one at a time, and creates the record.
- "We've got a new departure for [client] in [destination]." Same flow, framed as a new departure rather than a brand-new client.

## Not in this plugin

The day-by-day itinerary and the client-facing PDF, `tinto-client-itinerary-pdf`. Requesting and holding rooms with a hotel, and converting a sold hold into `Room Blocks` capacity, `tinto-batch-book-hotel-rooms`, this plugin only reads that once it exists. Taking a tour live for online booking, `tinto-booking-pages`. Entering a brand-new client's own record and branding, `tinto-add-winery-record`. All separate installs in the same marketplace.
