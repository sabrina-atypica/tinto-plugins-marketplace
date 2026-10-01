---
name: client-itinerary-pdf
description: >
  This skill should be used when Peter or Nélia says "create a new itinerary
  PDF for [tour/winery]," "we're about to sell [tour]," "we're trying to
  sell [tour]," "put together [guest]'s trip itinerary," "make the
  day-by-day schedule for [tour]," or otherwise needs the specific, dated,
  client-branded itinerary document for a tour, whether it's still being
  pitched, newly confirmed, or already sold, in the style of the
  client-supplied source itinerary PDFs (e.g. the "Fox Run Vineyards
  presents..." style document).
metadata:
  version: "0.12.0"
---

# Client Itinerary PDF

Produce the itinerary document for one **specific tour**: dated, with the real suppliers/hotels confirmed for that departure, a cover page of overview/pricing/inclusions, and the day-by-day schedule, as a branded PDF. **The primary recipient is the selling client (the winery, alumni association, etc.), not Tinto.** They use it to sell the tour and pass it on to their own prospective guests, so the document has to be sale-ready and correctly branded before it goes anywhere, not just accurate. Match the structure and tone of Tinto's existing client-supplied source itinerary PDFs (e.g. `2027 02 22 Fox Run alentejo portugal.pdf`), not a plain day-by-day list.

**This document gets built through a short conversation, not from a single request.** The flow below is the whole point of this skill: it exists so nothing gets guessed or inferred that should have been asked. Follow it every time, in order, even when the request already seems to include some of the answers: confirm rather than assume, because a wrong winery, a wrong price, or an unconfirmed itinerary on a guest-facing PDF is a real error, not a formatting detail.

## The conversational flow: do this every time, in order

This applies whether the tour is "about to sell," "trying to sell," or "already sold", all three are this skill, and all three go through the same steps. Ask one question, wait for the answer, then ask the next. Don't front-load a form, and don't build the PDF until Step 4.

### Step 1: Which winery/client?

If the winery or client hasn't been explicitly named, ask who it is before doing anything else. Don't infer it from an attached file, a filename, or what seems likely from context. Even when it seems obvious, confirm it out loud, the same way you'd confirm before writing anything to Airtable.

Once you have a name, look it up against the `Client / Affiliation` table (search by name, don't assume a fuzzy match is the right one; if two records look similar, ask which one).

- **Found an existing record** → note it and go to Step 3.
- **No matching record** → this is a new client. Go to Step 2 before continuing.

### Step 2: New winery, hand off to `tinto-add-winery-record`, don't redo its work here

Sourcing a new client's logo and brand color and filing them to Airtable is `tinto-add-winery-record`'s whole job, already built, maintained, and verified there. This skill doesn't keep its own second copy of that research-and-write process. Instead:

1. Check whether `tinto-add-winery-record`'s `add-winery-record` skill is available in this session (it's a separate, independently-installable plugin in the same marketplace, not bundled with this one).
2. **If it's available**, invoke it with the Skill tool, passing the confirmed client name and a note that this is for a new tour's itinerary PDF, so its branding is needed now rather than at Nélia's own pace. Wait for it to finish, then read back the `Client / Affiliation` record it created before moving to Step 3. Don't re-ask the questions that skill already asks (its own web-search-then-confirm flow for logo/color, its own commission/discount/payment-terms boundary); that's its job, not this skill's.
3. **If it isn't available in this session**, say so plainly: this plugin needs `tinto-add-winery-record` installed alongside it to handle a brand-new client, and ask Peter/Nélia to either install it or add the client's Airtable record themselves before continuing. Don't fall back to reimplementing the web-search-and-file steps here: a second, drifting copy of that logic is exactly what this hand-off is meant to avoid.

### Step 3: Confirm the tour details, one at a time

With the client identified (found directly in Step 1, or just created via `tinto-add-winery-record` in Step 2), ask the following one at a time: ask, get an answer, then move to the next. Don't list all five in one message.

1. **Date(s)** of the departure.
2. **Location / destination.**
3. **Price per person and single supplement** (see the note below for where these numbers come from, whether or not a `Hotel Rooms Booked w/ Hotel`/`Packages` already exist for this tour).
4. **Max capacity.**
5. **Whether the itinerary on record is correct.** Look up the tour in `Tours` (matching on client + dates/location from the answers above).
   - **If nothing matches**, this is a genuinely new tour with no `Tours` record yet. Creating that record properly (price, capacity, dates, status, slug, and everything else a new tour needs) is `tinto-add-new-tour`'s job, not this skill's: check whether it's available in this session and, if so, invoke it with the Skill tool the same way Step 2 hands off to `tinto-add-winery-record`, passing the confirmed client, destination, and dates. Once it finishes, read back the `Tours` record it created and continue below. If it isn't available, say so plainly and ask whether to install it or add the `Tours` record directly in Airtable first; don't reimplement full tour intake here.
   - **Once a `Tours` record exists (found directly, or just created via `tinto-add-new-tour`), check whether its destination has a reference file** (`references/destination-itineraries/<destination-slug>.md`) before touching `Itinerary Days` at all. See "Voice and narrative color" below: for most destinations this is the fastest, most accurate source, reused across every client there, not something to draft fresh or pull from an unrelated tour. Show its day-by-day, ask whether it's still accurate or needs updates, and treat the confirmed result (corrected as needed) as this departure's day-by-day for the PDF. This skill only reads `Itinerary Days`, it never writes to it, so if the confirmation surfaces a correction the stored record needs, flag it for a human to apply in Airtable rather than writing it from here.
   - **If `Itinerary Days` already has real, previously-confirmed content for this specific departure**, show that instead and ask for confirmation or corrections the normal way, and use the confirmed version for the PDF. This skill never overwrites `Itinerary Days`, so any correction is a human-owned Airtable edit, not something to write from here.
   - **If the destination has no reference file yet and `Itinerary Days` is empty**, propose a draft day-by-day for the PDF and get explicit sign-off before treating it as final. Never write the draft to `Itinerary Days`, this skill only reads that table; filling an empty record is a separate, human-owned Airtable edit.

   **While showing each day, also ask if there's any personal detail or story worth adding**, something about a guide, a winemaker, a chef, a specific moment, that the concise `Itinerary Days` copy wouldn't carry but this PDF can. `Itinerary Days` is deliberately terse (see "Voice and narrative color" below for why), so this is the one point in the flow where real, firsthand color can be captured directly from the person who actually has it, rather than left out or invented later.

Only once all five are confirmed does this move to Step 4. This skill only reads Airtable, it never writes: if a confirmed answer differs from what's stored (price, capacity, a day's description), use the confirmed value for the PDF and say plainly that the Airtable record is now out of date and needs a human to correct it (or a hand-off to the skill that owns that field), rather than writing the correction from here or silently building the PDF from the stale stored value.

### Step 4: Build the PDF

Only now, with the client branding, all five tour details, and any offered narrative color confirmed, generate the document. The rest of this skill (below) covers how.

## Not the same thing as the Pre/Post-Tour Planning Guide

Tinto also has a separate `tinto-prepost-tour-guide` skill that builds a **destination-level** reference guide (packing tips, arrival logistics, car rental, "surprises & tips," etc.), evergreen content that's the same for every guest going to that destination, not tied to one tour's specific dates or price. This skill is different: it's the **tour-specific sales/confirmation document** (cover page + Day 1: pickup at X, winery visit at Y, lunch at Z...) for one actual departure, sold under one specific client brand. A guest typically receives both documents at different points in the journey; don't conflate the two, and don't try to build one from the other. If a request is ambiguous about which is wanted, ask.

## Branding: this is client-branded, not Tinto-branded

Tinto sells tours white-labeled under the selling client's own brand (a winery, an alumni association, a wine shop, the `Client / Affiliation` table). **This document should carry that client's branding, the same way the booking pages do**, not Tinto's own burgundy/gold identity by default.

1. Use the `Client / Affiliation` record confirmed in Step 1, or just created via `tinto-add-winery-record` in Step 2: its logo and brand accent color, not the plain-text `Client / Affiliation` field on `Tours` (only the linked record is reliable, per the same distinction the `booking-page-publishing` skill checks).
2. Use the client's logo and accent color for the cover page and running header, in place of Tinto's own. Keep body typography consistent (see `references/tinto-brand.md` for the fallback type system) unless the client record specifies otherwise.
3. **If the confirmed client record still has no logo/color on file** (shouldn't happen after Step 2, but check), fall back to Tinto's own brand (`references/tinto-brand.md`) rather than shipping an unbranded or broken-looking cover page, and flag it, since that's also a booking-page gap worth fixing at the source.
4. A small "in partnership with Tinto Travels" or equivalent footer credit is appropriate even on a client-branded document; check an existing client-branded source PDF or ask for the exact convention if none is on hand.

## Voice and narrative color: the destination's reference itinerary is the reusable template, not one client's

**Most tours to the same destination run an identical or near-identical itinerary: the same wineries, the same restaurants, the same day order, just under a different client's brand.** `references/destination-itineraries/<destination-slug>.md` holds one real, previously-sent itinerary per destination, and is the reusable template for every tour there, not scoped to whichever client it was originally sent to. Its own frontmatter names that original client purely for provenance, so it's traceable to a real source document, not as a restriction on reusing it.

Use it this way:

1. **For any tour at a destination that already has a reference file, show its day-by-day to Peter/Nélia and ask directly whether it's still accurate, or whether anything needs updating** (a different guide, a closed restaurant, a swapped winery, a changed order). Don't silently assume it applies unchanged; confirm every time, since a real detail can shift even at a destination Tinto has sold before.

   **Show it as the condensed, booking-page-style version, explicitly framed as such, not as a preview of the finished PDF's writing.** Say plainly, in words to this effect: "Here's the high-level itinerary as it'll show on the corresponding booking page, is this correct?", and note that the actual PDF text will be the same full, marketing-style prose Tinto's itinerary PDFs already use (the reference file's own writing), not this condensed summary. Skipping this framing risks the condensed version reading like a downgrade from Tinto's usual style, when it's just the wrong layer to judge that from.
2. **Once confirmed, with any corrections applied, that becomes this departure's real day-by-day content for the PDF's fuller narrative prose.** This skill doesn't write a condensed version back to `Itinerary Days`, it only reads that table; keeping `Itinerary Days` in its concise, booking-page-shared style is a separate, human-owned edit. There's no separate "expand terse copy into narrative voice" step needed on top of this: the reference file's own prose, corrected as needed, is the narrative content directly. Only the client's own branding, not Fox Run's or whichever client the reference file names, goes on the cover.
3. **If `Itinerary Days` already holds real, previously-confirmed content for this specific departure**, that stays authoritative as-is. If it genuinely differs from the reference file, ask which is right rather than assuming either one, since a previously-confirmed departure's own record could reflect a deliberate, real difference, not staleness, and use the confirmed answer for the PDF. This skill never overwrites `Itinerary Days`; reconciling the stored record is a human-owned Airtable edit.
4. **Never invent a specific personal or anecdotal detail that isn't already confirmed.** A detail like "the winemaker lived 30 years in Hong Kong" or "the chef might get a Michelin star, he's only 30" is real color the reference file or Peter/Nélia supplied, not something to fabricate by extrapolation, even when it would make the copy read better. Draw color only from the reference file's confirmed text, or from whatever Peter/Nélia offer when asked in Step 3.
5. **If no reference file exists yet for this tour's destination** (a genuinely new destination Tinto hasn't sold before, or one like Coastal Tuscany that doesn't yet have its own reference distinct from Southern Tuscany & Umbria), say so and ask Peter/Nélia directly for the day-by-day and tone, or ask whether the nearest destination's reference file is close enough to use with extra caution. Don't silently default to a mismatched reference or fabricate a day-by-day without flagging that the destination has no reference yet.

## Where the data comes from

**Primary source of truth: the production Airtable base**, and by this point in the flow, the specific values confirmed in Step 3. Pull the tour's own header and cover-page details from the `Tours` record:

- `Overview`: the narrative summary for the cover page. Like `Itinerary Days`, this field is terse and shared with the booking page; apply the same voice-expansion approach as the day-by-day content above, using the destination's reference file's cover copy as the model, rather than reproducing it verbatim.
- `Price per Person`: a direct currency field on `Tours`, confirmed in Step 3. If the field is already populated, most often because `tinto-add-new-tour` captured it when the tour was created (the governed flow is to create the Planning/Proposed tour there first, then build this PDF), show that number back to Peter/Nélia and ask them to reconfirm it's still correct rather than skipping the question, since prices can change between a pitch and a later PDF for the same tour. If the field is genuinely unset (a tour created outside that flow), look up the Double-occupancy precedent from the most recent existing tour at the same destination, the same lookup `tinto-add-new-tour`'s Step 6 uses, to anchor the question, then ask Peter/Nélia directly, naming the source tour and its figure. Use the confirmed figure on the PDF, but don't write it to `Price per Person` from here, this skill only reads Airtable; setting that field is `tinto-add-new-tour`'s job (or a human edit), so say plainly the field is still unset and needs filling there.
- `Single Supplement`: read the **`Single Supplement (headline)`** single-line text field on `Tours`, the pitch/marketing figure `tinto-add-new-tour` captures when the tour is created (a single value or a range like `$700 to $1,200`). In the governed flow it's already there, so show it back to Peter/Nélia for reconfirmation and use it on the cover page as written. This skill only reads it, it never writes it.
  - **If a clean Double/Solo `Packages` pair already exists** for this tour (created by `tinto-add-new-tour`'s own Step 6 once a `Hotel Rooms Booked w/ Hotel` was in place), that per-room Double-minus-Solo gap is the **authoritative** figure, prefer it over the headline field, and if the two disagree, flag it so the headline can be refreshed. This is the common case for a PDF built after the tour is actually sold.
  - **If `Single Supplement (headline)` is blank** (a tour created outside the governed flow, or a genuinely undecided pitch price) and no `Packages` pair exists yet, don't just omit the line by default. Look up the same-destination precedent's single supplement, the gap between its Double and Solo `Packages` rows, to anchor the question, then ask Peter/Nélia directly, naming the source tour and its figures. Use the confirmed figure on this PDF's cover page. Don't write it to `Tours` from here, this skill only reads; say plainly the headline field is still empty and should be filled via `tinto-add-new-tour` (or by a human). Never create a `Packages` row for this: that stays reserved for a real, `Hotel Rooms Booked w/ Hotel`-backed offering, created only by `tinto-add-new-tour`'s Step 6 once the tour is actually sold. If Peter/Nélia would rather not commit to a number yet, a genuinely undecided pitch-stage price, it's fine to leave the line off and say so plainly, but that should be their call to make, not the default just because the field happens to be empty.
- `What's Included` / `What's Not Included`: cover-page inclusions list.
- `Max Participants`: a direct number field on `Tours`, confirmed in Step 3; include it on the cover page the way the source itinerary PDFs do ("Limited to N participants") if it's populated.

Then pull the day-by-day content from `Itinerary Days`, filtered to the requested tour by its linked `Tour` field, ordered by `Day #`. This is the version confirmed in Step 3, whether that came from an existing, previously-confirmed record or from the destination's reference file per "Voice and narrative color" above. Each row carries `Title` and `Description`; use the confirmed narrative content from Step 3 for the PDF's actual prose rather than this terse version, which stays what the booking page shows. (`Itinerary Days` has no time-of-day field; that granularity only exists on the operational `Bookings (Confirmations)`/`Standard Itineraries` tables, not here.)

**Don't trust a `Notes` field's description of what's populated over the actual field values.** Notes can go stale (e.g. a migration note saying a field was "left blank" after that field was since filled in); always read the live field, not what a note says about it.

**Apply Tinto's standing no-em-dash, no-gratuity-mention copy rules to `What's Included`/`What's Not Included` and `Overview` even if the stored text doesn't yet follow them.** That cleanup (see `whats-included-accommodation-audit.md`) was only applied to a specific 12-tour batch so far. Plenty of tours, including ones with real cover-page data, still have raw em dashes or a "gratuity included" mention in these fields. Rewrite on the way into the PDF (comma, colon, or sentence split in place of an em dash or "--"; delete gratuity mentions outright rather than rephrasing around them) rather than reproducing the field verbatim, and mention that the source field itself is still due for the same cleanup.

## Document structure

- **Cover page**: client logo/branding, tour title and dates, one-paragraph overview (expanded per "Voice and narrative color" above), price per person (and single supplement if determinable), what's included / not included. Guest name(s) only if producing a personalized copy for one reservation (check whether one shared PDF for the whole departure or a personalized one per guest is wanted; ask if unclear).
- **Day-by-day section**: one entry per day, in order: day number, title, expanded narrative description.
- **Closing section**: Tinto contact information / emergency contact, matching the convention used in Tinto's other guest-facing documents.

## Branding assets and layout

Build the PDF using the pdf skill (read its SKILL.md before generating). See `references/tinto-brand.md` for the fallback palette/typefaces and general layout conventions (same brand system used for the Pre/Post-Tour Planning Guide). Apply the client override from the Branding section above on top of that base layout, swapping logo and accent color rather than redesigning the whole document per client.

## Language and currency

Same rule as every other guest-facing surface: American English, no exceptions by destination. Pricing is USD. If the day's content includes a `Regional Sign-Off`-style destination-language flourish, that's a deliberate touch, not something to translate further or remove.

## After building

**Show the full draft to Peter or Nélia for feedback before treating the PDF as final.** The voice-expansion step above is a rewrite, not a transcription, so it needs a human check even when every underlying fact came straight from confirmed Airtable data or the reference file. Ask plainly whether anything reads off, whether any day needs a correction or an added detail, and regenerate before delivering the finished version.

Deliver the finished PDF the normal way. If this itinerary is being produced to attach to a `daily-guest-communications` touchpoint draft, say so plainly and remind whoever's sending the email that it still needs to be attached by hand; the Gmail draft itself never carries it.

## Escalate rather than guess

If any step in the conversational flow above can't get a clear answer (the winery name is ambiguous, `tinto-add-winery-record` isn't installed and no one's confirmed how to proceed, a tour detail conflicts with what's already in Airtable, `Itinerary Days` content still looks like a placeholder after being shown for confirmation, or the destination has no reference file yet), stop and ask rather than proceeding with a best guess. A guest reading a wrong, unbranded, missing-day, or fabricated-detail document is a real trip-planning and brand failure, not just a formatting issue, and it's cheaper to ask twice than to fix it after the PDF has gone out.
