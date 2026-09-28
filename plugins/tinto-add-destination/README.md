# Tinto Add Destination

Creating the `Destinations` table row that several `Tours` fields fall back to, a standalone plugin closing a dead end `tinto-add-new-tour` used to hit when a genuinely new destination came up.

## Overview

`Destinations` (`Hero Image(s)`, `Gallery`, `Overview Map SVG`, `Overview Map Caption`, `Hotel Name`/`Description`/`Photos`, `Itinerary Photos`, `Overview`, `What's Included`) is a real, load-bearing fallback table, a `Tours` record whose own value for any of these is blank inherits from its `Destinations` row, but until now no skill in this marketplace wrote to it. `tinto-add-new-tour` would stop cold when a named destination genuinely wasn't one of the known ones, and say adding the row was "a bigger decision than this skill should make alone," leaving it to someone working directly in Airtable.

New destinations come up rarely, roughly one (Coastal Tuscany) across the period this project has tracked, so this is a low-frequency skill by design, but it turns a genuine dead end into a real, if occasional, path.

## Components

| Skill | Purpose |
|---|---|
| `add-destination` | Confirms and files a new destination: creates the `Destinations` row with `Location` set to the exact string `Tours.Location` will use (matched by plain text, not a linked field, so exact spelling matters); drafts `Overview`/`What's Included` from a reference itinerary or a direct ask; web-sources `Hero Image(s)`/`Gallery` like `tinto-add-winery-record` sources a logo, but flags licensing/usage rights explicitly since these are guest-facing; offers an existing hotel supplier's details as the default for `Hotel Name`/`Description`/`Photos` where one exists; leaves `Overview Map SVG`/`Caption` alone as a manual cartography task. |

No agents or hooks, enforcement is already handled by Airtable seat permissions, not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** connector, and web search (for Hero Image(s)/Gallery only).

## Usage

Ask Claude things like:
- "We're launching tours in the Douro Valley now, can you set that up in Airtable?" Claude confirms the exact `Location` string, drafts an overview from what's given, and web-sources candidate hero photography with the source and licensing status shown before anything gets written.
- Or it fires automatically as a hand-off from `tinto-add-new-tour`, the moment a destination named there turns out to be genuinely new rather than one of the existing ones.

## Not in this plugin

Deciding whether Tinto should run tours to a new destination at all, that's Sales/Business Development, a separate process. Creating the actual `Tours` record for a departure there (`tinto-add-new-tour`'s job), adding a new choice to `Tours.Location`'s own dropdown (left to `tinto-add-new-tour`'s typecast write when the first real tour there is created, so a dropdown choice never exists without a real tour behind it), and `Overview Map SVG`/`Caption` (stays a manual design task).
