---
name: add-destination
description: >
  This skill should be used when anyone asks to "add a new destination," "we're
  launching tours in [place]," "set up [destination] in Airtable," or when
  tinto-add-new-tour hands off because a named destination genuinely isn't one
  of the known ones yet.
metadata:
  version: "0.2.0"
---

# Adding a Destination Record

Create the `Destinations` table row (base `appV1n8sUb3f60U7E`, table `tblhJEu58gSOwGNxW`) that several `Tours` fields fall back to when a tour's own value is blank: `Hero Image(s)`, `Gallery`, `Overview Map SVG`, `Overview Map Caption`, `Hotel Name` / `Hotel Description` / `Hotel Photos`, `Itinerary Photos`, `Overview`, and `What's Included`. This is ordinary data entry for a destination decision Peter (or whoever owns that call) has already made, not a sales/business-development action, the same boundary every other intake skill in this marketplace draws.

Since 2026-09-28 the row also carries `Arrival City` and four seasonal `Packing / Weather Tip` fields (Spring, Summer, Fall, Winter). These aren't a `Tours` fallback, the Mid-Trip guest email reads them directly from `Destinations` as required guest-facing content in their own right, and a blank cell there produces a visibly degraded draft rather than a hard failure. This skill fills all five at creation, see Step 2 below.

New destinations come up rarely (roughly one, Coastal Tuscany, across the period this project has tracked), so this fires far less often than `tinto-add-supplier` or `tinto-add-new-tour`, but until now `tinto-add-new-tour` hit a genuine dead end here: it would stop and tell whoever was asking that adding the `Destinations` row was "a bigger decision than this skill should make alone," with no path forward except doing it by hand in Airtable. This skill is that path.

## Scope check first

This skill covers filing a destination Tinto has already decided to run tours to. It does not cover deciding whether Tinto should launch tours somewhere, that's Sales/Business Development (BD), the same boundary `tinto-add-supplier` and `tinto-add-winery-record` draw around their own scope. If a request sounds like it's actually about evaluating a new destination rather than filing one already decided, say so and point it to Peter/Roger rather than treating it as data entry.

## Step 1: Confirm the destination and create the row

Confirm the destination's name/region with whoever's asking; per `tinto-add-new-tour`'s existing caution here, this is a real, deliberate decision worth a moment's confirmation, not something to wave through on a first mention.

**Dedupe first.** Check the `Destinations` table for an existing row with the same or a very similar name before creating a new one.

**The `Location` field is plain text, not a linked field**, and it's matched against `Tours.Location` by exact string, not by any relationship Airtable enforces. Set it to exactly the string that will be used in `Tours.Location` for tours at this destination (confirm the exact spelling/wording with whoever's asking, since this string is what actually connects the two tables). Getting this wrong silently breaks every fallback field on every future tour at this destination, with no error to catch it.

**On `Tours.Location`'s own dropdown**: this skill does not add a new choice to it. The Airtable connection available here can update a select field's name or description but not its list of choices, and there's a better reason to leave it alone anyway: `tinto-add-new-tour` writes the first real `Tours` record at a genuinely new destination with `typecast: true`, which lets Airtable add the matching dropdown choice itself at the moment a real tour actually uses it. That's deliberately the same moment, not sooner, since a choice with no tour behind it is exactly the kind of stray dropdown value (`Douro`, `Vinho Verde`) already flagged as a data-quality problem elsewhere in this base.

## Step 2: Arrival City and seasonal packing tips

Immediately after creating the row, fill five more fields that the Mid-Trip guest email reads directly, not a `Tours` fallback, but required guest-facing content in its own right: `Arrival City`, and four `Packing / Weather Tip` fields, one each for Spring, Summer, Fall, and Winter. The Email Templates table already has a fallback for a blank cell here, but it produces a visibly degraded draft, so fill all five at creation rather than leaving them for later.

**Arrival City** (plain text) is the one city guests actually fly into for this destination, the guest-facing airport city, not necessarily the tour's own base or its drop-off point. Alentejo's is Lisbon; Austria's is Munich even though that tour itself drops off in Vienna. Confirm this with whoever's asking rather than assuming it matches the destination name or the tour's own pickup location. `Tours.Pickup / Drop-off Location` stays the tour-specific answer for logistics; this field is the simpler, guest-facing one.

**The four seasonal Packing / Weather Tip fields** are short, warm, conversational guest-facing copy, not a checklist. Read a few of the 40 existing entries first, any handful of destinations already in this table, for voice before drafting new ones: one to two sentences, a lowercase start is fine since these get spliced into an email sentence, degrees Fahrenheit first (the guest base is US-heavy), and destination-specific rather than generic, naming the actual things that matter there (cobblestones, vineyard afternoons, altitude swings, a sea breeze, a swimsuit for a boat or catamaran day) rather than generic travel advice that could describe anywhere.

Draft all four seasons even when this destination's own tours currently only run in one or two of them. Per Sabrina's explicit call on this (2026-09-28): roughly 80 percent-accurate seasonal content drafted now beats a blank cell that forces someone to improvise at the moment a Mid-Trip email actually needs it. Draft from general climate knowledge for the region, and say plainly in the summary that these were auto-drafted for Nélia's review, not confirmed local knowledge; she (or her own session) can refine the wording later, directly in the cell, and the next email draft simply picks up whatever's current there.

Season mapping, for reference: Spring runs roughly March through May, Summer June through August, Fall September through November, Winter December through February.

## Step 3: Overview and What's Included

Before asking, make explicit what these two fields actually feed: the `Overview` and `What's Included` sections of this destination's own booking page, word for word, not internal notes. Whoever's answering needs to know that upfront, otherwise they don't know what register or length to answer in.

Always show a real example alongside the ask, not just a description of what's needed in the abstract: pull another destination's actual on-file `Overview` and `What's Included` copy (Alentejo is a good default) and show it verbatim as a model, for example "here's the exact copy we have for Alentejo's pages, what do you want for [destination]'s", rather than asking someone to write from a blank prompt.

Check whether a destination reference itinerary already exists (`tinto-client-itinerary-pdf`'s `references/destination-itineraries/`, if that plugin is installed in this session). If one does, offer its cover copy as a starting point, adapted rather than copied verbatim, and confirmed before writing, the same discipline `tinto-add-new-tour` uses for its own cover-page content. If nothing exists yet, ask Peter/Nélia directly for the destination's highlights, offer a rough draft itinerary suggestion when it helps (for example, notable wine regions worth including, given how many nights the tour runs), and draft `Overview`/`What's Included` from their answer. Propose, don't write silently.

## Step 4: Hero Image(s) and Gallery

Before asking, make explicit what these two fields actually are: exactly the hero photo and the gallery images that will show on this destination's own booking page, not general reference photos. All photos in both fields need to be horizontal (landscape) orientation, since that's what the booking page layout expects; say so plainly when asking.

**Ask first whether Peter, Nélia, or whoever's asking already has their own photography for this destination.** If yes, ask specifically for a Dropbox link, not a generic "send me a folder," since Dropbox is the tool Tinto actually uses for this. A Dropbox shareable link works directly: Airtable attachment fields are populated by handing Airtable a URL it fetches itself, not by direct file upload, so a Dropbox link works but a file that exists only on someone's own computer, or one attached directly in a chat message, has no public URL yet and can't be attached until it's somewhere with a shareable link.

**If they don't have their own photos**, search an explicitly free or royalty-free stock site (Unsplash or similar), not a general web search of the destination's tourism-board site or similar. Unlike a client's logo (`tinto-add-winery-record`'s job), these images go straight onto a live, public, guest-facing booking page, a materially different risk than internal branding, so:

1. Confirm any candidate is actually the current place, not a stock photo of somewhere else, an unrelated location, or an outdated view. A caption or tag claiming a location is not enough on its own: reject a photo whose claimed location isn't visually corroborated by the image itself, even when the metadata says otherwise.
2. Note the source explicitly and flag anything whose licensing or usage rights for this kind of commercial, public use aren't clear, rather than treating "found on the web" as good enough on its own.
3. Once a confident candidate is found, write it to the record (`Hero Image(s)` or `Gallery` as appropriate) rather than only describing it and waiting for approval first, then share the direct Airtable record link (below) in chat so Peter/Nélia can actually review the photos in place.
4. If nothing confidently usable turns up, leave the field blank and say so plainly, rather than using a questionable source to avoid an empty field.

## Step 5: Hotel Name, Hotel Description, Hotel Photos

Only once at least one `Suppliers` record of `Type` = Hotel is linked to a tour at this destination does this step have anything real to offer. Look up which hotel is most commonly used for tours there and offer its name, description, and photos as the default source, confirmed before writing, never invented. For a genuinely brand-new destination with no supplier on file yet, leave all three blank and say so plainly, this is a real gap to flag, not one to fill with placeholder content.

## Step 6: Overview Map SVG and Overview Map Caption

These are generated, not hand-drawn: every existing map in this table was built by Claude from real public geographic boundary data, in a consistent house visual style, not commissioned cartography. This skill builds them too, as its final step.

1. **Ask for, or derive from the conversation, the destination's base city or cities.** A destination can have more than one base, for example a tour that splits time between two towns; get the actual base or bases from whoever's asking rather than assuming there's exactly one.
2. **Fetch real public boundary data** for the destination's own region and its immediate neighbors, for context (for example `click_that_hood`'s per-country/region GeoJSON collections, or `johan/world.geo.json` for a whole neighboring country). Match the destination to the region it actually corresponds to in that dataset.
3. **Simplify the geometry** (for example with `shapely`'s `simplify`, tolerance around 0.03 degrees, `preserve_topology=True`) and drop small disconnected fragments, islands, slivers, below roughly 5 percent of the largest part's area, so the result matches the existing maps' stylized, low-poly look rather than raw high-resolution coastline data.
4. **Project into the established house style**: `viewBox="0 0 410 560"`, a `#4A1528` background rectangle, a muted `map-ctx` group for neighboring/context geography, a highlighted gold `map-dest` group for the destination's own region, `locmap-ctx-label` and `locmap-dest-label` text for country names, and a `locmap-pin-label` plus a two-circle pin marker (`r="9"` outline, `r="4.5"` filled, `#C9A84C`) for each base city. Base the projection window on the destination region's own bounding box plus a fixed margin, not on the full extent of every context shape, so the crop doesn't skew or zoom out to include something like a far-off island chain.
5. **Place a clearly labeled, non-overlapping pin for every base city.** When two pins sit close together, stagger their labels, one above, one below, rather than letting them collide.
6. **Render a preview** (for example with `cairosvg`) and show it before writing, the same confirm-before-write discipline every other field in this skill follows. Propose a caption in the same short style as the existing records, destination name and region in a few words, for example "The Alentejo, south-central Portugal."
7. Once confirmed, write both `Overview Map SVG` and `Overview Map Caption` to the record.

## Fallback for what wasn't sourced

Tell the person plainly which fields weren't filled and why (not yet decided, nothing reliable found, or still awaiting review in the case of auto-drafted packing tips), and send them a direct link to the record itself: `https://airtable.com/appV1n8sUb3f60U7E/tblhJEu58gSOwGNxW/<recordId>`.

## What's out of bounds here

**Deciding whether to run tours to this destination at all** is Sales/BD, covered under Scope check first above.

**Creating the actual `Tours` record** for a departure to this destination is `tinto-add-new-tour`'s job, not this skill's; this skill only creates the standalone `Destinations` row. If this skill was invoked as a hand-off from `tinto-add-new-tour` mid-flow, confirm the row's created and return control to it rather than continuing further.

**Suppliers, Hotel Rooms Booked w/ Hotel, Packages** and everything else downstream of an actual tour existing stay exactly where they already live (`tinto-add-supplier`, `tinto-batch-book-hotel-rooms`, `tinto-add-new-tour`'s own Step 6).

## After entering

Confirm back to the person exactly what was added and to which record: `Location` (the exact string), `Arrival City` and the four seasonal `Packing / Weather Tip` entries (flagging plainly that these were auto-drafted from general climate knowledge and need Nélia's review, not confirmed local knowledge), whatever got written for `Overview` / `What's Included`, whether Hero Image(s)/Gallery came from the client's own Dropbox or were web-sourced (with source and any licensing caveat), whether a Hotel Name/Description/Photos default was found or left blank, and the base city or cities plus the generated Overview Map SVG/Caption. Note the direct Airtable link for anything left incomplete or awaiting review.
