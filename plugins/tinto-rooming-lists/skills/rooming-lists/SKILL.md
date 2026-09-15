---
name: rooming-lists
description: >
  This skill should be used when Tamara asks to "create a rooming list,"
  "put together the room list for [tour]," "who's in which room for
  [tour]," "send the hotel our rooming list," or needs a per-tour breakdown
  of which guests occupy which rooms to send to a hotel or use for
  logistics planning.
metadata:
  version: "0.2.0"
---

# Rooming Lists

Produce a per-tour rooming list — which guests are booked into which room, at which property — as a spreadsheet Tamara can send to a hotel or use internally. This is a reporting task built from confirmed bookings, not a booking-management or capacity-planning tool: see "What this skill does not do" below before promising anything beyond the list itself.

## Where the data comes from

**Primary source of truth: the production Airtable base.** For the requested tour, filter `Reservations` directly by its `Tour` link field — every Reservation links straight to its Tour, confirmed live, so there's no need to go through `Packages` to find them. Each linked Reservation represents **one room actually booked, not one traveler** — a Double-occupancy reservation is still one room, with two guests in it, not two rows.

**Guest names come from the linked `Participants` records, never from the Reservation itself.** A Reservation's own `Buyer First Name`/`Buyer Last Name` fields hold a human-typed purchaser name, and in practice that field is often two people, a nickname, or an aside all run together — "Jennifer & Michael Howse," "William (Bill) & Chris Connor," "Dave & Kathy Muenz (Complimentary)" are all real examples. Don't parse guest names out of it. Every Reservation links to one `Participants` record per person actually traveling in that room, each with clean, separate First Name and Last Name fields — that's the real source for the sheet. The Buyer field is booking/contact metadata, useful for context, not for the guest-name column.

For each booked room, gather:

- Guest name(s) in that room, one per linked Participant record (a Double room has two, a Solo room has one).
- Property Name and Room Type (from the linked `Packages` row).
- Occupancy Type (Double / Solo).
- Bed Configuration (Double Bed / Twin Beds), if set — as of 2026-09-15 this field is blank on at least some destinations' `Packages` records (confirmed on the Alentejo tours). If it's blank, show the cell as blank and say so rather than guessing a configuration from Occupancy Type or leaving something that reads like an accidental omission.
- A room number, if one exists. There's no structured room-number field anywhere in Airtable, but Reservations are commonly hand-labeled in their own `Notes` field as they're taken — "Room 1.", "Room 2.", "Room 6." is the real, confirmed convention on at least one live tour. Check each Reservation's Notes for a `Room N.` pattern at the start and pull the number if it's there. If a Reservation's Notes has no such marker, leave the room-number column blank — don't invent a sequence or assume booking order matches room order; a wrong invented number is worse than an honest blank.
- Tour name and dates, for the spreadsheet header.

**Also worth surfacing when populated, not just assumed absent:** each `Participants` record can carry `Room Request Tags` and `Room Request Notes` — a stated room preference (twin vs. double, ground floor, adjoining rooms, and so on). Pull these per participant and add them as a plain notes column when any exist for the tour; this is exactly the kind of detail a hotel or Tamara would want on the sheet, not internal-only color. `Participants.Food Restriction` also exists on the same table — don't fold it into the rooming list by default, since sending dietary/allergy data to an external party (a hotel) is a different disclosure decision than an internal note. If Tamara wants it included, confirm first whether the specific list is going externally or staying internal, and only add the column once that's clear.

## Building the spreadsheet

Once the data is gathered, use the xlsx skill to build the actual file — read its SKILL.md before creating the spreadsheet rather than hand-rolling formatting. One row per booked room, grouped or sortable by property, with a header block naming the tour and dates. Include the room number column even when it's blank for some or all rows — a partially-filled column is still useful, and its absence would look like the data was never checked. Keep it plain and functional: this is a working document Tamara sends externally or uses to sanity-check against a hotel's own list, not a branded guest-facing piece.

Deliver the finished spreadsheet the normal way (send the file, don't just describe it in chat).

## What this skill does not do

**Don't compute "rooms remaining" from the `Packages.Capacity (deprecated - use Room Block)` field** — it's marked deprecated for exactly the reason it used to cause trouble (Double and Solo rows for the same physical room block each carrying their own independent number, double-counting one pool of physical rooms). Capacity now lives in a dedicated `Room Blocks` table (one row per physical block, linked to both the `Tour` and its `Packages` rows, with a single shared `Capacity` number covering every Occupancy Type sharing that block) — confirmed live and correctly shared across a Double/Solo package pair on at least one real tour. That's the correct source if "rooms remaining" is ever asked for. Check live whether the requested tour's Room Block actually has a populated `Capacity` before trusting a remaining-rooms number; if it's blank or the tour has no linked Room Block yet, say so rather than falling back to the deprecated Packages field. This skill's core output — **what's actually booked** — comes from real Reservations either way and doesn't depend on either capacity field being right.

**Don't filter or flag by `Payment Status`.** Rooming lists are built close to a tour's start (about a week out is the normal timing), by which point a reservation still sitting at Pending or otherwise unconfirmed shouldn't exist — so this isn't something the skill needs day-to-day logic for. If a Reservation genuinely does show up Pending or unpaid this close to the tour, that's unusual enough to be worth a mention to Tamara (see Escalate below), not a filtering rule to build around.

If a request wants something beyond a rooms-and-guests list — room assignment changes, moving a guest between rooms, anything that writes back to Airtable — treat it as a normal booking-data edit (within Tamara's existing Airtable write scope for day-to-day records), not a special case of this skill, and confirm the change plainly before making it.

## Escalate rather than guess

If a tour has bookings with no clear room/property link, a Reservation missing a linked Participant, or Occupancy Type / Bed Configuration data that looks inconsistent with the guest count, flag it to Tamara rather than guessing — a wrong rooming list sent to a hotel is worse than a short delay while it's confirmed. Also flag: a Reservation's Notes containing a room marker that doesn't cleanly parse as `Room N.` (e.g. it names two rooms, or reads ambiguously) rather than guessing which one it means; and a Reservation still showing an unconfirmed Payment Status this close to the tour's start, since that's outside the normal pattern and worth Tamara's own check rather than either including or silently dropping it.
