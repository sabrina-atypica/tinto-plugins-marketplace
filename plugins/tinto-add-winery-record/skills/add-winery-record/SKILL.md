---
name: add-winery-record
description: >
  This skill should be used when anyone on the team, not only Nélia
  (Peter and others use it too), asks to "add a new client," "add a new
  client affiliation," "add a new winery," "create a winery record," "add
  branding for [client]," or gives a new client's name to file into
  Airtable's `Client / Affiliation` table.
metadata:
  version: "0.3.0"
---

# Adding a Client / Affiliation Record

Guide whoever is asking through entering a new client/affiliation's record and branding into Airtable's `Client / Affiliation` table (base `appV1n8sUb3f60U7E`, table `tbly2EXYNDRx4YOFq`). This is ordinary data entry, the same write scope as any other record its user maintains, **not** a sales or business development action. Anyone on the team can trigger this, not only Nélia.

This record feeds the booking-page generator directly. A tour's booking page reads its client's `Client Name`, `Logo`, and `Brand Accent Color` straight from what gets entered here, so a bad value here (a wrong logo, a malformed color, an internal-looking name) shows up on a real guest-facing page later. The requirements below reflect what that generator actually needs, not just what Airtable will technically accept.

## Scope check first

This skill covers filing in an **already-decided** client relationship: someone Peter or Roger has already agreed will work with Tinto. It does not cover deciding whether to bring on a new client, negotiating commission terms, or anything else upstream of that decision, that's Sales/Business Development (BD), a separate process. If a request sounds like it's actually about landing or evaluating a new client rather than filing one already agreed, say so and point it to Peter/Roger rather than treating it as a data entry task.

## Step 1: Get the client name, then interview for anything else missing

The table is broader than wineries by design: restaurants, wine shops, alumni associations, travel agencies, financial advisors, and brands all sit on it too, so don't assume every entry is a winery.

At minimum you need the **Client Name**. This is the primary field, and the booking-page generator renders it verbatim: the header logo's alt text, the wordmark fallback, the page title, the footer, and every `{client}` token on the page. It must be the clean, guest-facing name only, never an internal code, a date prefix, or a "DRY RUN"/test suffix. If the request doesn't include it, ask directly rather than guessing.

If the person already has other details ready (a brand accent color they want used, notes, etc.), take those as given rather than re-deriving them; only fall back to the web-search steps below for whatever they didn't supply.

**Dedupe first, every time.** Before creating a new record, check whether one with that name already exists. Tours link directly to this record, so creating a duplicate splits a client's tours, and their commission, across two separate records. If it exists, update that record rather than creating a second one.

## Step 2: Source the logo and brand color yourself, don't ask for a chat upload

**Never ask the person to upload or paste the logo into the chat.** The Airtable connection can only fill the `Logo` field (an attachment field) from a public image URL that Airtable fetches itself; it cannot take a file attached to this conversation and push it into Airtable. A chat upload is not a shortcut here, it simply does not work.

Instead:

1. Web search for the client's official website.
2. Find their logo there (site header, footer, press or media page, or favicon) as a direct, publicly reachable image URL.
3. Before setting the `Logo` field, validate what you found, not just that a URL exists:
   - Transparent background, PNG or SVG. The booking page's header has its own light background; a logo baked onto a white or colored box will look wrong sitting on it.
   - Horizontal/landscape orientation reads best (it renders about 56px tall on the page, width auto).
   - It has to be the actual, current client logo. A plain image search often surfaces the wrong mark, a low-resolution scan, or a watermarked stock version, so check it against the site's own branding before using it rather than just taking the first image result.
   - Keep it modest: roughly 800px wide or less, and 500 KB or less is plenty. Don't pull in an oversized source file.
   If what you found doesn't meet these, keep looking on the site itself before falling back to Step 3.
4. Identify the site's primary brand color and set `Brand Accent Color` to a valid CSS hex code (`#RRGGBB`), and only that. This field drives both the accent styling and the wordmark color when there's no logo, so a scraped color name or a loose phrase like "goldish beige" won't fall back gracefully, it will just render wrong. Validate or normalize whatever you find into hex before writing it; never store a color name or a prose description in this field.

If you can't find the website, can't find a logo that passes the checks above, or can't confidently identify one dominant brand color (for example a black and white or reversed white logo with no clear accent), leave that field blank rather than guessing or forcing a value through, and move to Step 3 for that field only.

## Step 3: Fallback for what you couldn't source

Tell the person plainly that automatic sourcing didn't work for that field, and send them a direct link to the record itself so they can upload or fill it in Airtable: `https://airtable.com/appV1n8sUb3f60U7E/tbly2EXYNDRx4YOFq/<recordId>` (opens straight to that client's record, with the `Logo` and `Brand Accent Color` fields right there). Don't just say "upload it to Airtable," give the direct link.

## Step 4: Wine club discount, only if it applies

Ask directly whether this client runs its own wine-club member discount (most don't). If yes, capture all three together: **Wine Club Discount Code**, **Wine Club Discount Amount**, and **Wine Club Sign-Up URL**. This is what turns on the discount field in the booking form for their tours.

Only record what the person operating this skill tells you here, the same as any other interview answer in Step 1. This is not an invitation to propose, estimate, or guess a code or an amount.

This is a narrow, deliberate carve-out from the boundary below: these three fields get filled from what the operator states directly, never invented, the same way Client Name or Notes would be if the person already had them ready.

## What's out of bounds here, even though the seat technically allows it

Commission rate and payment terms are financial terms tied to the client relationship (see the Partner/Affiliates project); even though the write scope could technically touch these fields, treat them as **not yours to set**. If a record needs a commission rate and it isn't already there, flag it to Peter rather than filling in a value, even a placeholder. Wine club discount fields are the one exception, covered in Step 4 above, since the booking page needs them and they're something the operator states directly rather than something you'd otherwise have to invent.

## After entering

Confirm back what was added and to which record, including whether the logo and brand color came from the web or are still pending a manual upload, whether wine club discount fields were set, and note anything you couldn't find a clear field for.
