---
name: add-supplier
description: >
  This skill should be used when anyone on the team, not only Tamara or
  Nélia (Peter uses it too), asks to "add a new supplier," "add a new
  hotel," "create a supplier record," "we're working with
  [hotel/restaurant/guide] now," "file this hotel in Airtable," or gives
  a new supplier's name to file into Airtable's `Suppliers` table.
metadata:
  version: "0.1.0"
---

# Adding a Supplier Record

Guide whoever is asking through entering a new supplier's record into Airtable's `Suppliers` table (base `appV1n8sUb3f60U7E`, table `tblrjANbxY5baOuJp`). This is ordinary data entry, the same write scope as any other record its user maintains, **not** a sales or business development action. Anyone on the team can trigger this, not only Tamara or Nélia.

Every other skill that touches `Suppliers` (`tinto-add-new-tour`, `link-tour-suppliers`, `batch-book-hotel-rooms`, `daily-supplier-communications`) explicitly refuses to create a record for a named supplier that doesn't exist yet, it just says so and stops. This is the skill that actually creates one.

## Scope check first

This skill covers filing in an **already-decided** supplier relationship: someone Tinto has already agreed to work with. It does not cover deciding whether to bring on a new supplier or negotiating rates/terms, that's Sales/Business Development (BD), a separate process. If a request sounds like it's actually about landing or evaluating a new supplier rather than filing one already agreed, say so and point it to Peter/Roger rather than treating it as a data entry task.

## Step 1: Name, type, region, dedupe

Ask for the supplier's name if not given.

**Dedupe first, every time.** Before creating a new record, check whether one with that name already exists in `Suppliers`. If it does, update that record rather than creating a second one; a duplicate splits a supplier's bookings, payments, and lead-time rows across two records.

Confirm `Type` against the table's live single-select choices, don't rely on this list without re-checking the schema at run time, since choices can change: Hotel, Winery, Restaurant, Winery / Restaurant, Cultural Site, Cultural Site / Garden, Tour Guides, Transport, Activity / Tour, Cooking Class, Entertainment, Olive Producer, Wine Bar.

Ask `Region`.

## Step 2: Interview for what nothing can source

Ask directly, one at a time, the same conversational discipline every other intake skill here uses, for the fields that real day-to-day operation actually depends on: **Email**, **Preferred Communication Channel** (Email, WhatsApp, Phone Call, or Other, confirm against the live choice list), **Language Preference**, and **Payment Terms**. These are real operational facts only Tamara, Nélia, or Peter would know, never guess or infer them.

**`Contact Name`, `Phone`, and `Comments` are a deliberate exception**: a base-wide field audit found no automated consumer reads any of the three yet. Don't actively prompt for them. If the person offers one anyway (a name, a number, an aside worth recording), file it, the same as any other detail volunteered unprompted, but don't hold up the interview asking for it.

**`Notes / Specialties`** is operator-stated free text too, the same boundary `add-winery-record` draws around its Wine Club fields: this skill doesn't try to source or draft it. Record only what the person operating this skill states directly, and only if they offer it.

## Step 3: Hotel Cancellation Policy, never web-sourced

For a `Type` = Hotel supplier, `Hotel Cancellation Policy` and `Cancellation Policy Confirmed Date` are real, load-bearing data: `tinto-batch-book-hotel-rooms` reads them to compute a confirmed hold's `Cancellation Deadline`, and Room Count Reconciliation's lead time is meant to run against the same figure. Getting either wrong costs real money, a released hold that should have stayed held, or a deadline missed entirely, so treat this field differently from everything else in this skill.

**Never web-search this field, and never write to it from anything other than a real, confirmed answer from Tamara, Nélia, or Peter.** A hotel's own public website almost always describes terms for an individual traveler booking one room through their own booking engine, not the negotiated group/tour-operator terms Tinto actually operates under, which are frequently only agreed by direct correspondence or contract and never published anywhere public. A plausible-looking public policy is not the same thing as the real one, and writing it here would make a wrong number look confirmed.

1. Ask directly: does Tamara/Nélia/Peter already have this hotel's cancellation policy confirmed from the hotel itself (an email, a contract, a direct conversation), not what's published on their website?
2. **If yes**, record the policy exactly as stated and set `Cancellation Policy Confirmed Date` to today.
3. **If not yet confirmed**, leave both fields blank and say so plainly in your confirmation back to the person, this is a real gap worth chasing with the hotel directly, not something to fill with a placeholder or a web-search result.

## Step 4: Hotels, web-source photos and a description, confirm before writing

For a `Type` = Hotel supplier, `Property Photos` and `Guest Description` are lower-stakes than the cancellation policy above, cosmetic rather than financial, so the same web-search-then-confirm pattern `add-winery-record` uses for a client's logo applies here:

1. Web search for the hotel's own official website.
2. For `Property Photos`: find real property photography (exterior, a room, a common area) as direct, publicly reachable image URLs. Check it's actually the current property, not a stock image, a rendering, or an outdated renovation, before using it.
3. For `Guest Description`: draft a short, factual paragraph from what the hotel's own site says about itself (location, style, standout features), not marketing copy invented from nothing.
4. Show what was found and its source, and get explicit confirmation before writing either field.
5. If nothing reliable is found, leave the field blank and hand back a direct link to the Airtable record for manual completion (Step 6 below), the same fallback `add-winery-record` uses for a logo it couldn't confidently source.

## Step 5: Payment Terms and other free-text fields

Record `Payment Terms` exactly as stated by whoever is operating this skill; never infer standard terms from what other suppliers of the same `Type` have on file.

## Step 6: Fallback for what you couldn't source

Tell the person plainly which fields weren't filled and why (not confirmed yet, nothing reliable found), and send them a direct link to the record itself: `https://airtable.com/appV1n8sUb3f60U7E/tblrjANbxY5baOuJp/<recordId>`.

## What's out of bounds here, even though the seat technically allows it

**`Supplier Payments`** is Finance's territory (Peter/Nina), per the org-wide money rule, this skill never creates or updates rows there, and that holds regardless of anything entered above.

**Linking this supplier to a specific tour** is `link-tour-suppliers`' or `tinto-batch-book-hotel-rooms`' job once the record exists, not this skill's. This skill only creates or updates the standalone `Suppliers` record.

**Negotiating rates or terms with a new supplier** is Sales/BD, covered under Scope check first above.

## After entering

Confirm back to the person exactly what was added or updated and to which record, including whether photos/description came from the web or are still pending, whether `Hotel Cancellation Policy` was recorded from a real confirmed source or left blank pending follow-up, and note anything else you couldn't find a clear field for.
