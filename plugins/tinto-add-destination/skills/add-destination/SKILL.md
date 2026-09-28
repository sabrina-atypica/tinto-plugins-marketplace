---
name: add-destination
description: >
  This skill should be used when anyone asks to "add a new destination," "we're
  launching tours in [place]," "set up [destination] in Airtable," or when
  tinto-add-new-tour hands off because a named destination genuinely isn't one
  of the known ones yet.
metadata:
  version: "0.1.0"
---

# Adding a Destination Record

Create the `Destinations` table row (base `appV1n8sUb3f60U7E`, table `tblhJEu58gSOwGNxW`) that several `Tours` fields fall back to when a tour's own value is blank: `Hero Image(s)`, `Gallery`, `Overview Map SVG`, `Overview Map Caption`, `Hotel Name` / `Hotel Description` / `Hotel Photos`, `Itinerary Photos`, `Overview`, and `What's Included`. This is ordinary data entry for a destination decision Peter (or whoever owns that call) has already made, not a sales/business-development action, the same boundary every other intake skill in this marketplace draws.

New destinations come up rarely (roughly one, Coastal Tuscany, across the period this project has tracked), so this fires far less often than `tinto-add-supplier` or `tinto-add-new-tour`, but until now `tinto-add-new-tour` hit a genuine dead end here: it would stop and tell whoever was asking that adding the `Destinations` row was "a bigger decision than this skill should make alone," with no path forward except doing it by hand in Airtable. This skill is that path.

## Scope check first

This skill covers filing a destination Tinto has already decided to run tours to. It does not cover deciding whether Tinto should launch tours somewhere, that's Sales/Business Development (BD), the same boundary `tinto-add-supplier` and `tinto-add-winery-record` draw around their own scope. If a request sounds like it's actually about evaluating a new destination rather than filing one already decided, say so and point it to Peter/Roger rather than treating it as data entry.

## Step 1: Confirm the destination and create the row

Confirm the destination's name/region with whoever's asking; per `tinto-add-new-tour`'s existing caution here, this is a real, deliberate decision worth a moment's confirmation, not something to wave through on a first mention.

**Dedupe first.** Check the `Destinations` table for an existing row with the same or a very similar name before creating a new one.

**The `Location` field is plain text, not a linked field**, and it's matched against `Tours.Location` by exact string, not by any relationship Airtable enforces. Set it to exactly the string that will be used in `Tours.Location` for tours at this destination (confirm the exact spelling/wording with whoever's asking, since this string is what actually connects the two tables). Getting this wrong silently breaks every fallback field on every future tour at this destination, with no error to catch it.

**On `Tours.Location`'s own dropdown**: this skill does not add a new choice to it. The Airtable connection available here can update a select field's name or description but not its list of choices, and there's a better reason to leave it alone anyway: `tinto-add-new-tour` writes the first real `Tours` record at a genuinely new destination with `typecast: true`, which lets Airtable add the matching dropdown choice itself at the moment a real tour actually uses it. That's deliberately the same moment, not sooner, since a choice with no tour behind it is exactly the kind of stray dropdown value (`Douro`, `Vinho Verde`) already flagged as a data-quality problem elsewhere in this base.

## Step 2: Overview and What's Included

Check whether a destination reference itinerary already exists (`tinto-client-itinerary-pdf`'s `references/destination-itineraries/`, if that plugin is installed in this session). If one does, offer its cover copy as a starting point for `Overview` and `What's Included`, adapted rather than copied verbatim, and confirmed before writing, the same discipline `tinto-add-new-tour` uses for its own cover-page content. If nothing exists yet, ask Peter/Nélia directly for the destination's highlights and draft from their answer. Propose, don't write silently.

## Step 3: Hero Image(s) and Gallery, web-sourced, licensing flagged explicitly

Web search for candidate photography of the destination, the same web-search-then-confirm technique `tinto-add-winery-record` uses for a client's logo. Treat this with more caution than a logo, though: these images go straight onto a live, public, guest-facing booking page, a materially different risk than internal branding. Before attaching anything:

1. Confirm it's actually the current place, not a stock photo, an unrelated location, or an outdated view.
2. Note the source explicitly and flag anything whose licensing or usage rights for this kind of commercial, public use aren't clear, rather than treating "found on the web" as good enough on its own.
3. Show what was found, its source, and any licensing caveat, and get explicit confirmation before writing either field.
4. If nothing confidently usable turns up, leave both fields blank and say so plainly, rather than using a questionable source to avoid an empty field. Ask Peter/Nélia to supply their own photography instead.

## Step 4: Hotel Name, Hotel Description, Hotel Photos

Only once at least one `Suppliers` record of `Type` = Hotel is linked to a tour at this destination does this step have anything real to offer. Look up which hotel is most commonly used for tours there and offer its name, description, and photos as the default source, confirmed before writing, never invented. For a genuinely brand-new destination with no supplier on file yet, leave all three blank and say so plainly, this is a real gap to flag, not one to fill with placeholder content.

## Step 5: Overview Map SVG and Overview Map Caption, out of scope

These are real, bespoke cartography today, hand-built per destination, and a templated or auto-generated version would likely look worse than what's already on file. This skill does not attempt either field. Say plainly that this stays a manual/design task, and hand back a direct link to the record (below) for whoever handles that work.

## Fallback for what wasn't sourced

Tell the person plainly which fields weren't filled and why (not yet decided, nothing reliable found, out of scope by design), and send them a direct link to the record itself: `https://airtable.com/appV1n8sUb3f60U7E/tblhJEu58gSOwGNxW/<recordId>`.

## What's out of bounds here

**Deciding whether to run tours to this destination at all** is Sales/BD, covered under Scope check first above.

**Creating the actual `Tours` record** for a departure to this destination is `tinto-add-new-tour`'s job, not this skill's; this skill only creates the standalone `Destinations` row. If this skill was invoked as a hand-off from `tinto-add-new-tour` mid-flow, confirm the row's created and return control to it rather than continuing further.

**Suppliers, Room Blocks, Packages** and everything else downstream of an actual tour existing stay exactly where they already live (`tinto-add-supplier`, `tinto-batch-book-hotel-rooms`, `tinto-add-new-tour`'s own Step 6).

## After entering

Confirm back to the person exactly what was added and to which record: `Location` (the exact string), whatever got written for `Overview` / `What's Included`, whether Hero Image(s)/Gallery came from the web (with source and any licensing caveat) or are still pending, whether a Hotel Name/Description/Photos default was found or left blank, and a reminder that Overview Map SVG/Caption stays a manual task. Note the direct Airtable link for anything left incomplete.
