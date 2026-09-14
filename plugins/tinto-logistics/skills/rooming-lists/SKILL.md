---
name: rooming-lists
description: >
  This skill should be used when Tamara asks to "create a rooming list,"
  "put together the room list for [tour]," "who's in which room for
  [tour]," "send the hotel our rooming list," or needs a per-tour breakdown
  of which guests occupy which rooms to send to a hotel or use for
  logistics planning.
metadata:
  version: "0.1.0"
---

# Rooming Lists

Produce a per-tour rooming list — which guests are booked into which room, at which property — as a spreadsheet Tamara can send to a hotel or use internally. This is a reporting task built from confirmed bookings, not a booking-management or capacity-planning tool: see "What this skill does not do" below before promising anything beyond the list itself.

## Where the data comes from

**Primary source of truth: the production Airtable base.** For the requested tour, pull the linked `Reservations` through that tour's `Packages` rows (or filter `Reservations` directly by `Tour`, whichever the live schema makes easier — check the current field names rather than assuming). Each linked Reservation represents **one room actually booked, not one traveler** — a Double-occupancy reservation is still one room, with two guests in it, not two rows.

For each booked room, gather:

- Guest name(s) in that room (a Double room may have two names; check the Reservation record for how the second guest's name is stored — it may be a separate field or embedded in notes, confirm live rather than assuming a fixed schema)
- Property Name and Room Type (from the linked `Packages` row)
- Occupancy Type (Double / Solo)
- Bed Configuration (Double Bed / Twin Beds — blank on Solo rows)
- Tour name and dates, for the spreadsheet header

## Building the spreadsheet

Once the data is gathered, use the xlsx skill to build the actual file — read its SKILL.md before creating the spreadsheet rather than hand-rolling formatting. One row per booked room, grouped or sortable by property, with a header block naming the tour and dates. Keep it plain and functional: this is a working document Tamara sends externally or uses to sanity-check against a hotel's own list, not a branded guest-facing piece.

Deliver the finished spreadsheet the normal way (send the file, don't just describe it in chat).

## What this skill does not do

**Don't compute "rooms remaining" from the `Packages.Capacity (deprecated - use Room Block)` field** — it's marked deprecated for exactly the reason it used to cause trouble (Double and Solo rows for the same physical room block each carrying their own independent number, double-counting one pool of physical rooms). As of 2026-09-14, capacity has moved to a dedicated `Room Blocks` table (one row per physical block, linked to both the `Tour` and its `Packages` rows, with a single shared `Capacity` number) — that's the correct source if "rooms remaining" is ever asked for. Check live whether the requested tour's Room Block actually has a populated `Capacity` before trusting a remaining-rooms number; if it's blank or the tour has no linked Room Block yet, say so rather than falling back to the deprecated Packages field. This skill's core output — **what's actually booked** — comes from real Reservations either way and doesn't depend on either capacity field being right.

If a request wants something beyond a rooms-and-guests list — room assignment changes, moving a guest between rooms, anything that writes back to Airtable — treat it as a normal booking-data edit (within Tamara's existing Airtable write scope for day-to-day records), not a special case of this skill, and confirm the change plainly before making it.

## Escalate rather than guess

If a tour has bookings with no clear room/property link, a Reservation missing a guest name, or Occupancy Type / Bed Configuration data that looks inconsistent with the guest count, flag it to Tamara rather than guessing — a wrong rooming list sent to a hotel is worse than a short delay while it's confirmed.
