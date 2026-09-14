---
name: guest-itinerary-pdf
description: >
  This skill should be used when Nélia asks to "create the itinerary PDF
  for [tour]," "put together [guest]'s trip itinerary," "make the
  day-by-day schedule for [tour]," or needs the specific, dated,
  client-branded itinerary document for a booked tour — the attachment
  referenced by the `daily-guest-communications` skill's trip-logistics
  touchpoints, in the style of the client-supplied source itinerary PDFs
  (e.g. the "Fox Run Vineyards presents..." style document).
metadata:
  version: "0.3.0"
---

# Guest Itinerary PDF

Produce the guest-facing itinerary for one **specific booked tour** — dated, with the real suppliers/hotels actually confirmed for that departure, a cover page of overview/pricing/inclusions, and the day-by-day schedule — as a branded PDF attachment. Match the structure and tone of Tinto's existing client-supplied source itinerary PDFs (e.g. `2027 02 22 Fox Run alentejo portugal.pdf`), not a plain day-by-day list.

## Not the same thing as the Pre/Post-Tour Planning Guide

Tinto also has a separate `tinto-prepost-tour-guide` skill that builds a **destination-level** reference guide (packing tips, arrival logistics, car rental, "surprises & tips," etc.) — evergreen content that's the same for every guest going to that destination, not tied to one tour's specific dates or price. This skill is different: it's the **tour-specific sales/confirmation document** (cover page + Day 1: pickup at X, winery visit at Y, lunch at Z...) for one actual departure, sold under one specific client brand. A guest typically receives both documents at different points in the journey — don't conflate the two, and don't try to build one from the other. If a request is ambiguous about which is wanted, ask.

## Branding: this is client-branded, not Tinto-branded

Tinto sells tours white-labeled under the selling client's own brand (a winery, an alumni association, a wine shop — the `Client / Affiliation` table). **This document should carry that client's branding, the same way the booking pages do** — not Tinto's own burgundy/gold identity by default.

1. Pull the tour's **linked** `Client` field on the `Tours` record (not the plain-text `Client / Affiliation` field — only the linked record is reliable, per the same distinction the `booking-page-publishing` skill checks). Read that Client record's logo and brand accent color.
2. Use the client's logo and accent color for the cover page and running header, in place of Tinto's own. Keep body typography consistent (see `references/tinto-brand.md` for the fallback type system) unless the client record specifies otherwise.
3. **If no Client is linked, or the linked record has no logo/color on file**, fall back to Tinto's own brand (`references/tinto-brand.md`) rather than shipping an unbranded or broken-looking cover page — and flag to Nélia that the tour has no usable client branding, since that's also a booking-page gap worth fixing at the source.
4. A small "in partnership with Tinto Travels" or equivalent footer credit is appropriate even on a client-branded document — check an existing client-branded source PDF or ask Nélia for the exact convention if none is on hand.

## Where the data comes from

**Primary source of truth: the production Airtable base.** Pull the tour's own header and cover-page details from the `Tours` record:

- `Overview` — the narrative summary for the cover page.
- `Price per Person` — a direct currency field on `Tours`. Use it as-is; the "lowest bookable room price" convention (see `booking-page-publishing`) only matters as a fallback for a tour whose own `Price per Person` is blank but which has a real price range across `Packages`.
- `Single Supplement` — also a direct currency field on `Tours`, not something to derive — use it directly rather than computing it from Double/Solo `Packages` rows. Only fall back to a Packages-based estimate if this field is blank and a clean Double/Solo pair exists; otherwise leave the single-supplement line off rather than guessing.
- `What's Included` / `What's Not Included` — cover-page inclusions list.
- `Max Participants` — a direct number field on `Tours`; include it on the cover page the way the source itinerary PDFs do ("Limited to N participants") if it's populated.

Then pull the day-by-day content from `Itinerary Days`, filtered to the requested tour by its linked `Tour` field, ordered by `Day #`. Each row carries `Title` and `Description` — already written in marketing voice — use this content directly rather than rewriting it, unless Nélia asks for a change. (`Itinerary Days` has no time-of-day field — that granularity only exists on the operational `Bookings (Confirmations)`/`Standard Itineraries` tables, not here.)

Check both tables directly for the specific tour rather than assuming it matches the destination's usual pattern — a given tour can diverge from the standard schedule or pricing once real bookings are in. If a day or a cover-page field looks like it's missing, or content looks like a placeholder rather than real copy, flag it to Nélia rather than inventing filler. **Don't trust a `Notes` field's description of what's populated over the actual field values** — Notes can go stale (e.g. a migration note saying a field was "left blank" after that field was since filled in); always read the live field, not what a note says about it.

**Apply Tinto's standing no-em-dash, no-gratuity-mention copy rules to `What's Included`/`What's Not Included` and `Overview` even if the stored text doesn't yet follow them.** That cleanup (see `whats-included-accommodation-audit.md`) was only applied to a specific 12-tour batch so far — plenty of tours, including ones with real cover-page data, still have raw em dashes or a "gratuity included" mention in these fields. Rewrite on the way into the PDF (comma, colon, or sentence split in place of an em dash or "--"; delete gratuity mentions outright rather than rephrasing around them) rather than reproducing the field verbatim, and mention to Nélia that the source field itself is still due for the same cleanup.

## Document structure

- **Cover page**: client logo/branding, tour title and dates, one-paragraph overview, price per person (and single supplement if determinable), what's included / not included. Guest name(s) only if producing a personalized copy for one reservation (check whether Nélia wants one shared PDF for the whole departure or a personalized one per guest — ask if unclear).
- **Day-by-day section**: one entry per day, in order — day number, title, description.
- **Closing section**: Tinto contact information / emergency contact, matching the convention used in Tinto's other guest-facing documents.

## Branding assets and layout

Build the PDF using the pdf skill (read its SKILL.md before generating). See `references/tinto-brand.md` for the fallback palette/typefaces and general layout conventions (same brand system used for the Pre/Post-Tour Planning Guide) — apply the client override from the Branding section above on top of that base layout, swapping logo and accent color rather than redesigning the whole document per client.

## Language and currency

Same rule as every other guest-facing surface: American English, no exceptions by destination. Pricing is USD. If the day's content includes a `Regional Sign-Off`-style destination-language flourish, that's a deliberate touch, not something to translate further or remove.

## After building

Deliver the finished PDF the normal way. If this itinerary is being produced to attach to a `daily-guest-communications` touchpoint draft, say so plainly and remind whoever's sending the email that it still needs to be attached by hand — the Gmail draft itself never carries it.

## Escalate rather than guess

If `Itinerary Days` or the cover-page fields for the requested tour are empty, clearly incomplete, or look like unconfirmed placeholder content, or the tour has no usable client branding, say so and ask Nélia rather than producing a PDF with gaps or invented content — a guest reading a wrong, unbranded, or missing-day document is a real trip-planning and brand failure, not just a formatting issue.
