---
name: add-winery-record
description: >
  This skill should be used when anyone on the team, not only Nélia
  (Peter and others use it too), asks to "add a new client," "add a new
  client affiliation," "add a new winery," "create a winery record," "add
  branding for [client]," or gives a new client's name to file into
  Airtable's `Client / Affiliation` table.
metadata:
  version: "0.2.0"
---

# Adding a Client / Affiliation Record

Guide whoever is asking through entering a new client/affiliation's record and branding into Airtable's `Client / Affiliation` table (base `appV1n8sUb3f60U7E`, table `tbly2EXYNDRx4YOFq`). This is ordinary data entry, the same write scope as any other record its user maintains, **not** a sales or business development action. Anyone on the team can trigger this, not only Nélia.

## Scope check first

This skill covers filing in an **already-decided** client relationship: someone Peter or Roger has already agreed will work with Tinto. It does not cover deciding whether to bring on a new client, negotiating commission terms, or anything else upstream of that decision, that's Sales/Business Development (BD), a separate process. If a request sounds like it's actually about landing or evaluating a new client rather than filing one already agreed, say so and point it to Peter/Roger rather than treating it as a data entry task.

## Step 1: Get the client name, then interview for anything else missing

The table is broader than wineries by design: restaurants, wine shops, alumni associations, travel agencies, financial advisors, and brands all sit on it too, so don't assume every entry is a winery.

At minimum you need the **Client Name**. If the request doesn't include it, ask directly rather than guessing. If the person already has other details ready (a brand accent color they want used, notes, etc.), take those as given rather than re-deriving them; only fall back to the web-search steps below for whatever they didn't supply.

Before creating a new record, check whether one with that name already exists (clients are entered once and reused across every tour they buy) so you don't create a duplicate.

## Step 2: Source the logo and brand color yourself, don't ask for a chat upload

**Never ask the person to upload or paste the logo into the chat.** The Airtable connection can only fill the `Logo` field (an attachment field) from a public image URL that Airtable fetches itself; it cannot take a file attached to this conversation and push it into Airtable. A chat upload is not a shortcut here, it simply does not work.

Instead:

1. Web search for the client's official website.
2. Find their logo there (site header, footer, press or media page, or favicon) as a direct, publicly reachable image URL, and set the `Logo` field to that URL.
3. Identify the site's primary brand color (the dominant color in their header, logo, or brand assets) and convert it to a hex code for the `Brand Accent Color` field.

If you can't find the website, can't find a usable logo image URL (some sites block automated fetching), or can't confidently identify one dominant brand color (for example a black and white or reversed white logo with no clear accent), leave that field blank rather than guessing, and move to Step 3 for that field only.

## Step 3: Fallback for what you couldn't source

Tell the person plainly that automatic sourcing didn't work for that field, and send them a direct link to the record itself so they can upload or fill it in Airtable: `https://airtable.com/appV1n8sUb3f60U7E/tbly2EXYNDRx4YOFq/<recordId>` (opens straight to that client's record, with the `Logo` and `Brand Accent Color` fields right there). Don't just say "upload it to Airtable," give the direct link.

## What's out of bounds here, even though the seat technically allows it

Commission rate, discount codes (`Wine Club Discount Code` / `Wine Club Discount Amount`), and payment terms are financial terms tied to the client relationship (see the Partner/Affiliates project); even though the write scope could technically touch these fields, treat them as **not yours to set**. If a record needs a commission rate or a discount code and it isn't already there, flag it to Peter rather than filling in a value, even a placeholder.

## After entering

Confirm back what was added and to which record, including whether the logo and brand color came from the web or are still pending a manual upload, and note anything you couldn't find a clear field for.
