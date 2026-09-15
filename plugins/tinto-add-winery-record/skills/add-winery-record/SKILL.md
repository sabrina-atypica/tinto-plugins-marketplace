---
name: add-winery-record
description: >
  This skill should be used when Nélia asks to "add a new winery," "create a
  winery record," "add branding for [winery]," "file this logo under
  [winery]," or hands over a logo file and/or brand colors for a winery
  client — entering an already-agreed winery client's record and branding
  into Airtable.
metadata:
  version: "0.1.0"
---

# Adding a Winery Record

Guide Nélia through entering a winery client's record and branding into Airtable. This is ordinary data entry — the same write scope as any other record she maintains — **not** a sales or business-development action.

## Scope check first

This skill covers filing in an **already-decided** client relationship: a winery Peter or Roger has already agreed will work with Tinto, sending Nélia their name, logo, and brand details to enter. It does not cover deciding whether to bring on a new client, negotiating commission terms, or anything else upstream of that decision — that's Sales/BD, a separate process that hasn't been built yet. If a request sounds like it's actually about landing or evaluating a new client rather than filing one already agreed, say so and point it to Peter/Roger rather than treating it as a data-entry task.

## What to enter

Look up the client record table in the production Airtable base (client/winery records may appear under a table named `Client / Affiliation` or `Wineries` depending on when the base was last checked — confirm the current name live rather than assuming). For a new or existing winery:

- Winery name
- Logo — if Nélia hands over a file in chat, attach it directly to the record's logo/attachment field
- Brand colors (hex codes), if provided
- Any other branding fields already on the record schema

Nélia can do this either by asking Claude to file the material she hands over, or by entering it directly in Airtable herself — both are her call.

## What's out of bounds here, even though the seat technically allows it

Commission rate, discount codes, and payment terms are financial terms tied to the client relationship (see the Partner/Affiliates project) — even though her Airtable write scope could technically touch these fields, treat them as **not hers to set**. If a winery record needs a commission rate or a discount code and it isn't already there, flag it to Peter rather than filling in a value, even a placeholder.

## After entering

Confirm back to Nélia what was added and to which record, and note anything you couldn't find a clear field for — an unclear schema match is worth a quick check rather than a guessed field.
