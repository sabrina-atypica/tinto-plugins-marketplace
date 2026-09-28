# Tinto Add Destination

Creating the `Destinations` table row that several `Tours` fields fall back to, a standalone plugin closing a dead end `tinto-add-new-tour` used to hit when a genuinely new destination came up.

## Overview

`Destinations` (`Hero Image(s)`, `Gallery`, `Overview Map SVG`, `Overview Map Caption`, `Hotel Name`/`Description`/`Photos`, `Itinerary Photos`, `Overview`, `What's Included`) is a real, load-bearing fallback table, a `Tours` record whose own value for any of these is blank inherits from its `Destinations` row, but until now no skill in this marketplace wrote to it. `tinto-add-new-tour` would stop cold when a named destination genuinely wasn't one of the known ones, and say adding the row was "a bigger decision than this skill should make alone," leaving it to someone working directly in Airtable.

Since 2026-09-28 the row also carries `Arrival City` and four seasonal `Packing / Weather Tip` fields that the Mid-Trip guest email reads directly, not a `Tours` fallback, but required guest-facing content in their own right.

New destinations come up rarely, roughly one (Coastal Tuscany) across the period this project has tracked, so this is a low-frequency skill by design, but it turns a genuine dead end into a real, if occasional, path.

## Components

| Skill | Purpose |
|---|---|
| `add-destination` | Confirms and files a new destination: creates the `Destinations` row with `Location` set to the exact string `Tours.Location` will use (matched by plain text, not a linked field, so exact spelling matters); fills `Arrival City` and four seasonal `Packing / Weather Tip` fields for the Mid-Trip email, auto-drafted for Nélia's review; drafts `Overview`/`What's Included` from a reference itinerary or a direct ask, always shown against a real example from an existing destination; web-sources `Hero Image(s)`/`Gallery` from free/royalty-free stock sites when the client has none of their own (asking for a Dropbox link when they do), flagging licensing and rejecting any photo whose claimed location isn't visually corroborated by the image itself; offers an existing hotel supplier's details as the default for `Hotel Name`/`Description`/`Photos` where one exists; generates `Overview Map SVG`/`Caption` from real public geographic boundary data in the established house style, with a labeled pin for each base city. |

No agents or hooks, enforcement is already handled by Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, and web search (for Hero Image(s)/Gallery and the Overview Map's boundary data).

## Usage

Ask Claude things like:
- "We're launching tours in the Douro Valley now, can you set that up in Airtable?" Claude confirms the exact `Location` string, fills Arrival City and the seasonal packing tips, drafts an overview against a real example from an existing destination, web-sources candidate hero photography with the source and licensing status shown before anything gets written, and generates the Overview Map from real boundary data.
- Or it fires automatically as a hand-off from `tinto-add-new-tour`, the moment a destination named there turns out to be genuinely new rather than one of the existing ones.

## Not in this plugin

Deciding whether Tinto should run tours to a new destination at all, that's Sales/Business Development, a separate process. Creating the actual `Tours` record for a departure there (`tinto-add-new-tour`'s job), and adding a new choice to `Tours.Location`'s own dropdown (left to `tinto-add-new-tour`'s typecast write when the first real tour there is created, so a dropdown choice never exists without a real tour behind it).
