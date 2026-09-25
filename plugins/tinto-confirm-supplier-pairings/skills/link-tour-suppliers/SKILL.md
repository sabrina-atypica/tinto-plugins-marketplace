---
name: link-tour-suppliers
description: >
  This skill should be used when Tamara asks to "link suppliers for [tour],"
  "set up the suppliers for the new tour," "do the handover for [tour],"
  "which tours still need suppliers linked," or when the daily supplier run
  (`daily-supplier-communications`) lists a tour as "sold, not yet linked to
  suppliers." It links a newly sold tour to its suppliers by copying the most
  recent tour at the same destination, after one confirmation from Tamara,
  writing both Bookings (Confirmations) and Supplier Booking Lead Times in the
  same step.
metadata:
  version: "0.3.1"
---

# Link a New Tour to Its Suppliers

This is the sales to ops handover. Peter or Nélia sells a tour and creates it with `tinto-add-new-tour`, which deliberately does not touch suppliers. Until someone links the tour to its suppliers, nothing else in the system knows who to contact: the daily supplier emails never fire, and the bus itinerary is empty. This skill does that linking.

The working assumption is simple: **a new tour to a destination uses the same itinerary and the same suppliers as the most recent tour to that destination.** Tamara confirms that once, here, and can swap individual suppliers before anything is written. If a supplier later turns out to be different for this tour, she corrects it when the daily run is about to draft to that supplier (see `daily-supplier-communications`, "When Tamara says a supplier is wrong for a tour"). Tinto no longer uses ops spreadsheets for this; Airtable is the only record.

## Where things live (production base `appV1n8sUb3f60U7E`)

| Table | ID | Fields used |
|---|---|---|
| Tours | `tblBQwWkdDq2VBGAI` | Tour Name, Location, Start Date, End Date, Status, Bookings (Confirmations), Supplier Booking Lead Times |
| Bookings (Confirmations) | `tbl26YiyYcNWt2Urv` | Activity Description, Day #, Date, Time Slot, Time, Stop Type, Location / Address Note, Logistics Note, Status, Notes, Supplier, Tour |
| Supplier Booking Lead Times | `tblXG1Nz4rJuDR9oY` | Label, Supplier Category, Booking Lead Time (Days Before Tour Start), Confidence, Notes, Booking Request Status, Tour, Supplier |
| Suppliers | `tblrjANbxY5baOuJp` | Supplier Name, Type, Email |

Bookings (Confirmations) is the day-by-day plan: one row per stop. Supplier Booking Lead Times is the contact schedule: one row per touchpoint per supplier (first contact, Final Reconfirmation, and for hotels three hotel touchpoints). They are not connected in Airtable, so this skill keeps them consistent by writing both at once.

## Step 1: find the tours that need linking

A tour needs linking when its `Status` is **Confirmed** and it has **no Bookings (Confirmations) rows at all**. List these with their Location and Start Date, soonest first, unless Tamara named one specific tour.

Also run the resume check: any tour with Bookings (Confirmations) rows carrying `[LINKED AT HANDOVER BY TAMARA` whose emailable, non-hotel suppliers have no Supplier Booking Lead Times rows for that tour. That means a previous linking run stopped halfway. Finish it (Step 5 only) rather than re-copying Step 4.

Before doing anything else for a tour, confirm it is not a duplicate of an existing departure (same client, destination and dates). If it might be, stop and ask.

## Step 2: pick the source tour

The source is **the most recent tour at the same destination** (same `Location`) that already has suppliers linked:

- It has Bookings (Confirmations) rows, and none of them start with `PRELIMINARY` (those rows came from a guest itinerary PDF only and were never verified).
- Its Status is Confirmed, In Progress or Completed (never Planning or Cancelled).
- "Most recent" means the latest `Start Date` among those tours. It can be a departure that is later in the year than the new one; that is fine, it is still the current version of the itinerary.
- Never the new tour itself.

Always say which tour you picked and why, by name and start date. If there are several recent candidates for the same destination and their supplier lists differ (this happens on Northern Adriatic & Slovenia, and some clients get their own variation, for example a different winery), show the difference and ask which one to copy.

If the destination has no suitable tour at all (a brand-new destination), stop. Say so, and ask Tamara how she wants to build the first one. The `Standard Itineraries` table can be offered as a rough starting point, but every row there is unconfirmed by design, so anything built from it must be flagged `PRELIMINARY` in Notes.

## Step 3: propose, and get one confirmation

Pull every Bookings (Confirmations) row of the source tour and show Tamara the new tour's day-by-day plan:

- For each row: Day #, new Date, Time Slot (and Time if set), Activity Description, Supplier.
- New Date = the new tour's Start Date + (Day # minus 1). Do not copy the source dates.
- Group by day so it reads like an itinerary, not a data dump.

Flag, before she answers:

- **Different length.** If the new tour's duration (End Date minus Start Date) differs from the source's, say which days would fall outside the new tour.
- **Different weekday.** If the new tour starts on a different weekday than the source, say so. Some destinations rotate suppliers by weekday (for example the Alentejo Wednesday and Saturday winery lunches), so the copy may put a supplier on a day it doesn't host groups.
- **Hotels.** Hotels are booked at the point of sale by `tinto-batch-book-hotel-rooms`. Check whether the new tour already has hotel rows in Supplier Booking Lead Times, and whether they are the same hotel(s) as the source's check-in rows. If the hotel differs, the Bookings row should use the new tour's actual hotel; say so and use that one.
- **Suppliers that aren't stops.** Some suppliers work behind the scenes and are not linked on any Bookings row, typically guides (for example the Alentejo Ducal Palace and São Cucufate guides) and sometimes transport. They only exist as `Supplier Booking Lead Times` rows on the source tour. List every non-hotel supplier that has rows on the source tour but no Bookings row, as a separate "also contacted" list, so Tamara confirms them too.
- **Stops with no supplier.** Rows with an empty Supplier are copied as they are (the plan still needs the stop), but list them so Tamara can fill any she knows.

Then ask one question: "Same itinerary and suppliers as <source tour>? Or tell me what's different." Accept any mix of: confirm all, swap the supplier on specific stops, drop a stop, add a stop. If she names a supplier that has no Suppliers record, do not create one here. Leave that stop's Supplier empty, note it, and point her to adding the supplier first.

Nothing is written until she has answered.

## Step 4: write Bookings (Confirmations)

Right before writing, re-check that the new tour still has zero Bookings (Confirmations) rows (another session may have linked it in the meantime). Then create one row per stop, linked to the new tour:

- Copy: Activity Description, Day #, Time Slot, Time, Stop Type, Location / Address Note, Logistics Note, Supplier (with Tamara's swaps applied).
- Date: recalculated as in Step 3.
- Status: `Pending`. Confirmation Date: empty. This is a planned stop, not yet confirmed with the supplier.
- Notes: `[LINKED AT HANDOVER BY TAMARA <today>, copied from <source tour name> (<source tour record ID>)]`. For a swapped stop, add `Supplier changed at handover: was <old>, now <new>.` Do not copy the source row's Notes; they describe the source tour's history, not this one.

Create in batches of up to 50 and check every batch's response count before moving on.

## Step 5: write Supplier Booking Lead Times

For each distinct supplier on the new tour's Bookings (Confirmations) rows, plus each confirmed supplier from the "also contacted" list in Step 3:

1. **Hotels:** skip. They belong to `tinto-batch-book-hotel-rooms`. If a hotel on the plan has no Supplier Booking Lead Times rows for this tour, flag it in the summary; do not create hotel rows here.
2. **No email on file:** skip (typical for cultural sites and some guides). There is nobody to email, so no touchpoint rows. List these in the summary.
3. **Already has rows for this tour:** skip, never duplicate.
4. **Otherwise, copy the touchpoints from the source tour.** If the source tour has Supplier Booking Lead Times rows for the same supplier, create the same set for the new tour: same Label suffix (for example ` (Final Reconfirmation)`), Supplier Category, Booking Lead Time and Confidence. This keeps any lead time Peter or Nina have already confirmed.
5. **If the source has no rows for that supplier** (typically a supplier Tamara swapped in), use the standard draft set for its type:

| Supplier Type | Supplier Category | First contact row | Final Reconfirmation row |
|---|---|---|---|
| Winery | Winery | 60 days | 28 days |
| Restaurant, Winery / Restaurant | Restaurant | 30 days | 28 days |
| Wine Bar | Wine Bar | 30 days | 28 days |
| Activity / Tour | Activity / Tour | 30 days | 28 days |
| Cultural Site | Cultural Site | 30 days | 28 days |
| Cooking Class | Cooking Class | 60 days | 28 days |
| Olive Producer | Olive Producer | 60 days | 28 days |
| Cultural Site / Garden | Cultural Site / Garden | 60 days | 28 days |
| Entertainment | Entertainment | 60 days | 28 days |
| Transport | Transport | 210 days | 28 days |
| Tour Guides | Tour Guides | 180 days | none |

   These rows get Confidence `Draft estimate — needs confirming` (exact option name). The lead time is a draft; the pairing itself is confirmed by the handover marker below.

For every row created:

- Label: `<Tour Name> - <Supplier Name>` plus the suffix, for example ` (Final Reconfirmation)`.
- Booking Request Status: `Not Due Yet`.
- Tour and Supplier linked.
- Notes: `[LINKED AT HANDOVER BY TAMARA <today>, copied from <source tour name>]`. The daily drafting gate accepts this marker, so these rows will draft when due. Add a line saying whether the lead time was copied from the source tour or is the standard draft for the category.

If a new row would already be due today or overdue (a tour sold late), say so in the summary: the next daily run will draft it immediately.

## Step 6: verify and report

Re-query both tables for the new tour and check the counts match what you created. Then report to Tamara, briefly:

- Source tour used, and any swaps she made.
- Bookings (Confirmations): number of stops created, and any stops left without a supplier.
- Supplier Booking Lead Times: number of rows created, suppliers skipped (hotels, no email) and why.
- Which touchpoint comes first and when (usually the transport or guide first contact), so she knows when the first email for this tour will be drafted.
- Anything flagged along the way (length or weekday differences, hotel mismatch, missing supplier records).

## What this skill does not do

- It does not change lead-time rules or add touchpoint types. Those are Peter and Nina's decisions. If a lead time looks wrong, note it in the row's Notes as `[TIMING FLAGGED BY TAMARA <date>: <reason>]` and say it needs their decision.
- It does not create Suppliers records, hotel rows, Itinerary Days (guest-facing copy, owned by `tinto-client-itinerary-pdf`) or anything on the Tours record.
- It does not re-link tours that already have Bookings (Confirmations) rows, including the tours still marked `PRELIMINARY`. Those are left as they are for now (Sabrina, 2026-09-25).
- It never sends or drafts supplier emails. That is `daily-supplier-communications`.

## Escalate rather than guess

- No suitable source tour, or several candidates that disagree and Tamara isn't sure which is right.
- A possible duplicate tour.
- A supplier she names that isn't in the Suppliers table.
- Any write that returns fewer records than sent: stop, re-query, and report exactly what exists before retrying, so nothing is created twice.
