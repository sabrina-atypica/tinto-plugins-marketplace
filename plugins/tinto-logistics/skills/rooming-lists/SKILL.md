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

**Do not report "rooms remaining" or compute capacity math from the `Capacity` field on Packages.** That field is known to be unreliable as of 2026-09-10: Double and Solo rows for the same physical room block are modeled as two independent rows, each with its own Capacity number, when in reality they share one pool of physical rooms (a booked Double and a booked Solo can be eating into the same 12 rooms, not 12 Doubles + 12 Solos = 24). At least one live example (Boordy Vineyards) has this set incorrectly. If Tamara asks for "how many rooms are left," don't compute it from this field — tell her the Capacity data has a known accuracy issue and point her to Sabrina/Peter/Nina rather than producing a number that might be wrong. This skill only reports **what's actually booked**, which comes from real Reservations and doesn't depend on the flawed Capacity field at all — that part is reliable.

If a request wants something beyond a rooms-and-guests list — room assignment changes, moving a guest between rooms, anything that writes back to Airtable — treat it as a normal booking-data edit (within Tamara's existing Airtable write scope for day-to-day records), not a special case of this skill, and confirm the change plainly before making it.

## Escalate rather than guess

If a tour has bookings with no clear room/property link, a Reservation missing a guest name, or Occupancy Type / Bed Configuration data that looks inconsistent with the guest count, flag it to Tamara rather than guessing — a wrong rooming list sent to a hotel is worse than a short delay while it's confirmed.
