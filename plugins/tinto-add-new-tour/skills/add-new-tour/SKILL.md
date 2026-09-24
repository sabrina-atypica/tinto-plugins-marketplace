---
name: add-new-tour
description: >
  This skill should be used when Peter or Nélia says "add a new tour," "create
  a new tour for [client]," "we've got a new departure for [client] in
  [destination]," "set up [destination] for [client]," "record this new tour
  in Airtable," or otherwise needs a brand-new `Tours` record created for a
  tour that doesn't exist in Airtable yet.
metadata:
  version: "0.1.0"
---

# Add New Tour

Create the `Tours` record for a tour Tinto has already agreed to run. This is ordinary data entry for a decision Peter (or whoever owns the client relationship) has already made, not a sales or business-development action, and it doesn't cover deciding whether to run a tour, only recording one that's real.

Every tour currently in Airtable arrived through a one-time spreadsheet migration in September 2026, not through an ongoing intake process. This skill is that process: a lot depends on a `Tours` record actually being complete (rooming lists, bus-company itineraries, supplier-pairing audits, booking pages, both daily-communications skills all read from it), so guessing or leaving fields blank here causes real downstream problems, not just an incomplete-looking Airtable row.

**This document gets built through a short conversation, not from a single request**, the same reasoning as `tinto-add-winery-record` and `tinto-client-itinerary-pdf`: ask one question, wait for the answer, then ask the next, and confirm rather than assume even when the request already seems to answer some of it.

## The conversational flow: do this every time, in order

### Step 1: Which client?

If the client hasn't been explicitly named, ask before doing anything else. Look it up against the `Client / Affiliation` table (search by name, don't assume a fuzzy match is the right one; if two records look similar, ask which one).

- **Found an existing record** → note it and go to Step 3.
- **No matching record** → this is a new client. Go to Step 2 before continuing.

### Step 2: New client, hand off to `tinto-add-winery-record`, don't redo its work here

Sourcing a new client's logo and brand color and filing them to Airtable is `tinto-add-winery-record`'s whole job. Don't keep a second copy of that research-and-write process here.

1. Check whether `tinto-add-winery-record`'s `add-winery-record` skill is available in this session.
2. **If it's available**, invoke it with the Skill tool, passing the confirmed client name and a note that this is for a new tour. Wait for it to finish, then read back the `Client / Affiliation` record it created before moving to Step 3.
3. **If it isn't available**, say so plainly and ask Peter/Nélia to either install it or add the client's Airtable record themselves before continuing. Don't reimplement the web-search-and-file steps here.

### Step 3: Confirm the tour's identity, one at a time

With the client identified, ask the following one at a time, not as a form. Check for a duplicate before treating anything as settled: a client can have more than one tour to the same destination (different departures), and a destination can have more than one client. Match on client **and** destination **and** dates together, never on client or destination alone.

1. **Destination.** Must be a real one: Alentejo, Porto & Douro, Southern Tuscany & Umbria, Coastal Tuscany, Loire Valley, Castille & León, Northern Adriatic & Slovenia, Peloponnese, Puglia, or Austria (a real, sold, one-off destination, not on the public marketing site). The `Location` field's dropdown also still has two stray leftover choices, "Douro" and "Vinho Verde", that aren't real destinations, used by nothing, and shouldn't be picked. If the destination named genuinely isn't one of the ten above, stop and ask Peter/Nélia to confirm it's real before adding a new `Location` choice: the `Destinations` table (which several other fields fall back to) needs its own row for a genuinely new destination, and that's a bigger decision than this skill should make alone.
2. **Start and end dates.**
3. **Status**: Planning, Confirmed, In Progress, Completed, or Cancelled. Ask which applies now rather than defaulting; a tour that's still being negotiated is Planning, one the client has committed to is Confirmed.
4. **Price per person.** This is required. `Price per Person` feeds the booking page directly and renders as a literal broken price if left blank, so don't create the record without it. If it genuinely isn't set yet, say so and ask whether to hold off creating the record until it is, rather than writing a placeholder.
5. **Max participants.**
6. **Pickup / drop-off location** (e.g. "Lisbon → Lisbon").
7. **Cover-page content**: a short `Overview` paragraph, `What's Included` / `What's Not Included`, and, if available, `Tour Highlights` and `Hero Facts`. See the conventions below for how to draft a starting point to confirm rather than asking for all of this from scratch.
8. **Anything else worth recording as `Notes`** right now (a hotel on hold, a data source, a caveat) that doesn't fit a structured field.

There is no dedicated `Single Supplement` field on `Tours` (removed 2026-09-22: no clean single value existed for multi-hotel destinations). Don't create one or write to one. Single-supplement pricing lives on the `Packages` table instead, as the price difference between a Double and a Solo row for the same room, and that table is out of scope for this skill (see below).

### Step 4: Generate the Tour Name and Slug, confirm before writing

**Tour Name**: the established convention for an existing tour is "{Client}: An Insider's View of {Region}, the {Destination}" (using a colon rather than the em dash some older records use, per Tinto's standing no-em-dash copy rule) for a client-branded tour, or "Tinto Presents the Joys of {Region}" for a self-branded Tinto departure with no named client. Propose the fitting one, but confirm before treating it as final. Skip the "{Region}, the" clause where it reads awkwardly for the destination (e.g. Puglia, Austria).

**Slug**: generate from `{location-slug}-{client-slug}-{year}-{month}` (e.g. `alentejo-linganore-2027-10`), matching the pattern already used on existing tours. Check it doesn't collide with an existing `Slug` value before writing it; if the same client has more than one departure to the same destination in the same month, append a distinguishing suffix and confirm it with whoever's asking.

### Step 5: Create the record

Only once the destination, dates, status, price, capacity, pickup/drop-off, cover-page content, Tour Name, and Slug are all confirmed, create the `Tours` record with the linked `Client` (and a short plain-text mirror in `Client / Affiliation` for display parity, ask what short form to use if it isn't obvious). Leave `Publish Status` alone (defaults to Draft; that field belongs to the booking-page publish pipeline, not tour intake).

## Conventions for drafting cover-page content

- **What's Included / What's Not Included**: real, genuine per-destination boilerplate already exists on other confirmed tours at the same destination (accommodation tier, meal/tasting/transport inclusions), not a generic default. Look up an existing tour at the same destination and offer its inclusions list as the starting point to confirm or adjust, rather than drafting from nothing. Apply Tinto's standing no-em-dash, no-gratuity-mention house style regardless of what the source tour's own copy does.
- **Overview**: a short, factual, booking-page-style paragraph, the same terse register `Itinerary Days` and existing `Tours.Overview` values use, not the expanded narrative voice `tinto-client-itinerary-pdf` builds for the client-facing PDF later. Offer a draft based on the destination's known highlights (from an existing tour there, or from the client's own confirmed itinerary details if given) and confirm it rather than treating a first draft as final.
- **Hero Facts**: a specific pipe-delimited convention, one fact per line, `value | label`, for example:
  ```
  6 nights | Duration
  Évora | Home base
  Small group (up to 20) | Format
  Lisbon → Lisbon | Pick-up / drop-off
  ```
  Match this shape rather than free-form text.

## Not the same thing as the day-by-day itinerary or checkout setup

This skill creates the tour's identity, not everything a tour eventually needs.

**Day-by-day itinerary (`Itinerary Days`)**: deliberately out of scope. `tinto-client-itinerary-pdf` already has a confirm-and-draft flow for this (its Step 3, item 5), and duplicating that logic here would mean two places that can drift apart. After creating the `Tours` record, say plainly that the itinerary isn't built yet, and ask whether to hand off to `tinto-client-itinerary-pdf` right now (if it's available) to build it, or leave it for whenever someone next needs the client-facing PDF. Don't draft day-by-day content in this skill.

**Checkout inventory (`Packages`, `Room Blocks`, `Hotel Room Inventory`)**: also out of scope. Whether and when a tour goes live for online booking is a separate, later decision, not something that should happen automatically the moment a tour is recorded; several existing tours have sat with no `Packages` at all for months by deliberate choice. `tinto-batch-book-hotel-rooms` and `tinto-booking-pages` own that stage. Mention plainly that this step still needs doing before the tour could ever go live for booking, but don't build it here.

## Escalate rather than guess

If any step above can't get a clear answer, the client is ambiguous, `tinto-add-winery-record` isn't installed and no one's confirmed how to proceed, the named destination isn't one of the real ten, price per person isn't available yet, or a possible duplicate tour can't be ruled out from client + destination + dates alone, stop and ask rather than proceeding with a best guess. A `Tours` record with a wrong price, wrong destination, or a silent duplicate of an existing departure causes real downstream errors (a broken booking page, a supplier audit run against the wrong tour, a rooming list built for the wrong departure), not just a messy Airtable row.
