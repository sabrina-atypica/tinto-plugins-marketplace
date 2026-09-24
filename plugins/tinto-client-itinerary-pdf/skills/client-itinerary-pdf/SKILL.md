---
name: client-itinerary-pdf
description: >
  This skill should be used when Peter or Nélia says "create a new itinerary
  PDF for [tour/winery]," "we're about to sell [tour]," "we're trying to
  sell [tour]," "put together [guest]'s trip itinerary," "make the
  day-by-day schedule for [tour]," or otherwise needs the specific, dated,
  client-branded itinerary document for a tour — whether it's still being
  pitched, newly confirmed, or already sold — in the style of the
  client-supplied source itinerary PDFs (e.g. the "Fox Run Vineyards
  presents..." style document).
metadata:
  version: "0.6.0"
---

# Client Itinerary PDF

Produce the itinerary document for one **specific tour** — dated, with the real suppliers/hotels confirmed for that departure, a cover page of overview/pricing/inclusions, and the day-by-day schedule — as a branded PDF. **The primary recipient is the selling client (the winery, alumni association, etc.), not Tinto** — they use it to sell the tour and pass it on to their own prospective guests, so the document has to be sale-ready and correctly branded before it goes anywhere, not just accurate. Match the structure and tone of Tinto's existing client-supplied source itinerary PDFs (e.g. `2027 02 22 Fox Run alentejo portugal.pdf`), not a plain day-by-day list.

**This document gets built through a short conversation, not from a single request.** The flow below is the whole point of this skill: it exists so nothing gets guessed or inferred that should have been asked. Follow it every time, in order, even when the request already seems to include some of the answers — confirm rather than assume, because a wrong winery, a wrong price, or an unconfirmed itinerary on a guest-facing PDF is a real error, not a formatting detail.

## The conversational flow — do this every time, in order

This applies whether the tour is "about to sell," "trying to sell," or "already sold" — all three are this skill, and all three go through the same steps. Ask one question, wait for the answer, then ask the next. Don't front-load a form, and don't build the PDF until Step 4.

### Step 1 — Which winery/client?

If the winery or client hasn't been explicitly named, ask who it is before doing anything else. Don't infer it from an attached file, a filename, or what seems likely from context — even when it seems obvious, confirm it out loud, the same way you'd confirm before writing anything to Airtable.

Once you have a name, look it up against the `Client / Affiliation` table (search by name, don't assume a fuzzy match is the right one — if two records look similar, ask which one).

- **Found an existing record** → note it and go to Step 3.
- **No matching record** → this is a new client. Go to Step 2 before continuing.

### Step 2 — New winery: hand off to `tinto-add-winery-record`, don't redo its work here

Sourcing a new client's logo and brand color and filing them to Airtable is `tinto-add-winery-record`'s whole job, already built, maintained, and verified there. This skill doesn't keep its own second copy of that research-and-write process. Instead:

1. Check whether `tinto-add-winery-record`'s `add-winery-record` skill is available in this session (it's a separate, independently-installable plugin in the same marketplace, not bundled with this one).
2. **If it's available**, invoke it with the Skill tool, passing the confirmed client name and a note that this is for a new tour's itinerary PDF, so its branding is needed now rather than at Nélia's own pace. Wait for it to finish, then read back the `Client / Affiliation` record it created before moving to Step 3. Don't re-ask the questions that skill already asks (its own web-search-then-confirm flow for logo/color, its own commission/discount/payment-terms boundary) — that's its job, not this skill's.
3. **If it isn't available in this session**, say so plainly: this plugin needs `tinto-add-winery-record` installed alongside it to handle a brand-new client, and ask Peter/Nélia to either install it or add the client's Airtable record themselves before continuing. Don't fall back to reimplementing the web-search-and-file steps here — a second, drifting copy of that logic is exactly what this hand-off is meant to avoid.

### Step 3 — Confirm the tour details, one at a time

With the client identified (found directly in Step 1, or just created via `tinto-add-winery-record` in Step 2), ask the following one at a time — ask, get an answer, then move to the next. Don't list all five in one message.

1. **Date(s)** of the departure.
2. **Location / destination.**
3. **Price per person** (and single supplement, if there is one).
4. **Max capacity.**
5. **Whether the itinerary on record is correct.** Look up the tour in `Tours` (matching on client + dates/location from the answers above — if nothing matches, this is a genuinely new tour and there's no existing `Tours` record to check against, say so and treat every field, including the day-by-day, as new). If a `Tours` record exists, pull its linked `Itinerary Days` and show the day-by-day as currently stored, day by day if it's long, and ask for confirmation or corrections. If there's no itinerary on record yet, propose a draft day-by-day and get explicit sign-off before treating it as final — don't write a proposed itinerary to `Itinerary Days` without that confirmation.

Only once all five are confirmed does this move to Step 4. If any answer changes something already written to Airtable (price, capacity, a day's description), write that correction back to the `Tours`/`Itinerary Days` record as part of confirming it, and say plainly that you updated it — don't silently build the PDF from the old stored value alongside a new spoken one.

### Step 4 — Build the PDF

Only now, with the client branding and all five tour details confirmed, generate the document. The rest of this skill (below) covers how.

## Not the same thing as the Pre/Post-Tour Planning Guide

Tinto also has a separate `tinto-prepost-tour-guide` skill that builds a **destination-level** reference guide (packing tips, arrival logistics, car rental, "surprises & tips," etc.) — evergreen content that's the same for every guest going to that destination, not tied to one tour's specific dates or price. This skill is different: it's the **tour-specific sales/confirmation document** (cover page + Day 1: pickup at X, winery visit at Y, lunch at Z...) for one actual departure, sold under one specific client brand. A guest typically receives both documents at different points in the journey — don't conflate the two, and don't try to build one from the other. If a request is ambiguous about which is wanted, ask.

## Branding: this is client-branded, not Tinto-branded

Tinto sells tours white-labeled under the selling client's own brand (a winery, an alumni association, a wine shop — the `Client / Affiliation` table). **This document should carry that client's branding, the same way the booking pages do** — not Tinto's own burgundy/gold identity by default.

1. Use the `Client / Affiliation` record confirmed in Step 1, or just created via `tinto-add-winery-record` in Step 2 — its logo and brand accent color, not the plain-text `Client / Affiliation` field on `Tours` (only the linked record is reliable, per the same distinction the `booking-page-publishing` skill checks).
2. Use the client's logo and accent color for the cover page and running header, in place of Tinto's own. Keep body typography consistent (see `references/tinto-brand.md` for the fallback type system) unless the client record specifies otherwise.
3. **If the confirmed client record still has no logo/color on file** (shouldn't happen after Step 2, but check), fall back to Tinto's own brand (`references/tinto-brand.md`) rather than shipping an unbranded or broken-looking cover page — and flag it, since that's also a booking-page gap worth fixing at the source.
4. A small "in partnership with Tinto Travels" or equivalent footer credit is appropriate even on a client-branded document — check an existing client-branded source PDF or ask for the exact convention if none is on hand.

## Where the data comes from

**Primary source of truth: the production Airtable base**, and by this point in the flow, the specific values confirmed in Step 3. Pull the tour's own header and cover-page details from the `Tours` record:

- `Overview` — the narrative summary for the cover page.
- `Price per Person` — a direct currency field on `Tours`, confirmed in Step 3.
- `Single Supplement` — also a direct currency field on `Tours`, not something to derive — use it directly rather than computing it from Double/Solo `Packages` rows. Only fall back to a Packages-based estimate if this field is blank and a clean Double/Solo pair exists; otherwise leave the single-supplement line off rather than guessing.
- `What's Included` / `What's Not Included` — cover-page inclusions list.
- `Max Participants` — a direct number field on `Tours`, confirmed in Step 3; include it on the cover page the way the source itinerary PDFs do ("Limited to N participants") if it's populated.

Then pull the day-by-day content from `Itinerary Days`, filtered to the requested tour by its linked `Tour` field, ordered by `Day #` — this is the version confirmed in Step 3, including any corrections written back at that point. Each row carries `Title` and `Description` — already written in marketing voice — use this content directly rather than rewriting it, unless asked for a change. (`Itinerary Days` has no time-of-day field — that granularity only exists on the operational `Bookings (Confirmations)`/`Standard Itineraries` tables, not here.)

**Don't trust a `Notes` field's description of what's populated over the actual field values** — Notes can go stale (e.g. a migration note saying a field was "left blank" after that field was since filled in); always read the live field, not what a note says about it.

**Apply Tinto's standing no-em-dash, no-gratuity-mention copy rules to `What's Included`/`What's Not Included` and `Overview` even if the stored text doesn't yet follow them.** That cleanup (see `whats-included-accommodation-audit.md`) was only applied to a specific 12-tour batch so far — plenty of tours, including ones with real cover-page data, still have raw em dashes or a "gratuity included" mention in these fields. Rewrite on the way into the PDF (comma, colon, or sentence split in place of an em dash or "--"; delete gratuity mentions outright rather than rephrasing around them) rather than reproducing the field verbatim, and mention that the source field itself is still due for the same cleanup.

## Document structure

- **Cover page**: client logo/branding, tour title and dates, one-paragraph overview, price per person (and single supplement if determinable), what's included / not included. Guest name(s) only if producing a personalized copy for one reservation (check whether one shared PDF for the whole departure or a personalized one per guest is wanted — ask if unclear).
- **Day-by-day section**: one entry per day, in order — day number, title, description.
- **Closing section**: Tinto contact information / emergency contact, matching the convention used in Tinto's other guest-facing documents.

## Branding assets and layout

Build the PDF using the pdf skill (read its SKILL.md before generating). See `references/tinto-brand.md` for the fallback palette/typefaces and general layout conventions (same brand system used for the Pre/Post-Tour Planning Guide) — apply the client override from the Branding section above on top of that base layout, swapping logo and accent color rather than redesigning the whole document per client.

## Language and currency

Same rule as every other guest-facing surface: American English, no exceptions by destination. Pricing is USD. If the day's content includes a `Regional Sign-Off`-style destination-language flourish, that's a deliberate touch, not something to translate further or remove.

## After building

Deliver the finished PDF the normal way. If this itinerary is being produced to attach to a `daily-guest-communications` touchpoint draft, say so plainly and remind whoever's sending the email that it still needs to be attached by hand — the Gmail draft itself never carries it.

## Escalate rather than guess

If any step in the conversational flow above can't get a clear answer — the winery name is ambiguous, `tinto-add-winery-record` isn't installed and no one's confirmed how to proceed, a tour detail conflicts with what's already in Airtable, or `Itinerary Days` content still looks like a placeholder after being shown for confirmation — stop and ask rather than proceeding with a best guess. A guest reading a wrong, unbranded, or missing-day document is a real trip-planning and brand failure, not just a formatting issue, and it's cheaper to ask twice than to fix it after the PDF has gone out.
