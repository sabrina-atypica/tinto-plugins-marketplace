---
name: booking-page-publishing
description: >
  This skill should be used when Nélia or Tamara asks to "check if [tour]'s
  booking page is ready," "publish [tour]'s page," "why isn't [tour]
  bookable yet," or needs to audit and publish a tour's public sign-up/
  booking page.
metadata:
  version: "0.3.1"
---

# Booking Page Readiness and Publishing

Audit a tour's Airtable data against what its public booking page actually needs, then publish it. **Read the warning below before doing anything else — it materially changes what "safe to publish" means right now, as of 2026-09-14.**

## ⚠ Known live issue: guest payments may not be reaching Airtable

An audit on 2026-09-14 found that the pending-write step from a booking page's checkout flow into Airtable's `Reservations` table is **not currently working in production**, confirmed by the complete absence of any `TT-`prefixed booking reference in `Reservations`, including for a real completed test payment. **If this is still unresolved, a real guest who completes a payment on a live booking page today would have no Airtable record of it at all** — no reservation, no confirmation trail, nothing for the rest of the automation to act on.

**Before publishing any tour's page for the first time, or promoting a preview to live, confirm directly with Sabrina, Peter, or Nina that this has been fixed and verified with a real test transaction.** Do not treat the checklist below as "safe to publish" on its own — a page can pass every readiness item and still silently lose real bookings if this is still broken. If you can't get a clear confirmation that it's fixed, say so plainly and hold off on the publish step, even if asked to proceed — flag it rather than guess.

## Publishing does not need repo or GitHub access — a correction from an earlier version of this skill

An earlier version of this skill said publishing needed repo/deploy access most Cowork sessions don't have. **That's no longer true, as of 2026-09-14.** There are two publish endpoints on the live Worker, and only one of them has that requirement:

- `POST /api/publish-page` — the **old** path. Needs a tour's page HTML rendered locally first (`buildPageData`/`renderTourPage`, run against the booking-page repo), which does need repo access. This is what the earlier version of this skill was describing, and what `deploy-runbook.md` in this Project still documents (that doc is stale, not wrong — it predates the endpoint below).
- `POST /api/publish-tour` — the **current** path this skill uses. Send it a tour's record ID; the Worker itself reads the tour's data from Airtable, renders the page with the currently-deployed template, writes the result into that Tour's `Page HTML` field, and copies its images into R2. Nothing about this needs GitHub, a repo, or a device bridge — only Airtable access (to get the tour ID and the publish key) and the ability to make an HTTPS POST request, which every Cowork session's shell can already do.

Verified directly before writing this: `curl -X POST https://tintotravels.com/api/publish-tour` with no `X-Publish-Key` header returns `401 {"error":"Unauthorized: missing or incorrect X-Publish-Key header."}` rather than a 404 — confirming the route is live and auth-gated, not confirming behavior beyond that from this check alone.

**One caveat that still stands:** `/api/publish-tour` only renders with whatever page template is *currently deployed* on the Worker. If Sabrina or Peter is mid-way through a template/markup change via the repo, republishing a tour here will use the old template until that deploy lands — this skill doesn't know when a repo-side deploy is in flight, so if a published page looks visually stale after a known template change, that's the likely reason, not a bug in this flow.

## Readiness checklist

Pull the tour's record from the production Airtable base and check each of these directly — don't assume from a spreadsheet or prior conversation, the live base is the source of truth:

| Field | Requirement | Failure mode if missing |
|---|---|---|
| `Slug` | Set, unique | `/api/publish-tour` returns `400` — a page can't be published without one |
| `Price per Person` | Set (use the *lowest bookable room's* price if the tour has a price range — this is the existing convention, not a special case) | Renders literal "$NaN" on the hero |
| `Overview`, `Tour Highlights`, `What's Included` / `What's Not Included` | Populated, guest-facing copy | Blank sections on the page |
| Hero / Gallery photos | At least one, on the Tour or inherited from its `Destinations` record | Degrades gracefully to a standard gradient hero if blank — not blocking, but check before promoting externally |
| Linked `Client` (not the plain-text `Client / Affiliation` field) | Populated | Zero client branding (logo/accent color) even if the plain-text field looks filled in — these are two different fields, check the linked one specifically |
| Linked `Itinerary Days` | Populated, matches the tour's actual confirmed schedule | Guests see no day-by-day content, or content that doesn't match what they booked |
| Linked `Packages` | At least one row, `Live Status` = Bookable (not Pre-registration or another status) | Nothing for checkout to sell — page renders but guests can't book |

**Also check, not blocking but worth knowing:** "X rooms left" style messaging should read capacity from the `Room Blocks` table (one shared `Capacity` per physical room block, linked to the Tour and its `Packages` rows) — not from `Packages.Capacity (deprecated - use Room Block)`, which is deprecated precisely because it used to double-count Double/Solo rows sharing one physical block (see the `rooming-lists` skill in the Logistics plugin for detail). If the tour has no linked Room Block yet, or its Capacity is blank, don't show a remaining-rooms number at all rather than guessing.

## When a tour isn't ready: a specific suggestion for every gap, not a checklist of failures

A flat pass/fail report isn't the deliverable. **Every single gap gets a specific, concrete suggested value that Nélia only has to confirm or correct** — never a bare "this is missing" and never an open-ended "what should this be?" with nothing proposed. The work this skill does before reporting a gap is: check what real information is actually available for *this* tour first, then fall back to precedent/convention to fill in a specific candidate — always land on a proposal, calibrated by how confident that proposal actually is.

For each missing or incomplete checklist field:

1. **Check this tour's own record first** — `Notes`, any other free-text field, attachments, or related records (e.g. a linked Client's Notes) sometimes already contain the real answer (a confirmed rate, a decided property, a draft description) that just hasn't been copied into the structured field yet. This is the highest-confidence source — use it directly if it's there rather than falling through to precedent.
2. **If nothing tour-specific is on file, look for real precedent** — another tour to the same destination, especially one this tour's itinerary was copied from or closely follows — and propose that as the starting point.
3. **Present the proposal with its source, and ask for confirmation or correction, not permission to proceed.** E.g. "Slug would be `alentejo-vaux-hall-2027-10`, following the standard `{location}-{client}-{year}-{month}` pattern — correct, or does this need adjusting?" or "Tour Highlights would default to the same 5 points as [other tour], since this itinerary follows the same route — correct as-is, or does this tour need its own updates?" Don't gate this behind "should I write this?" — propose it as the default and only change course if Nélia corrects it.

**Packages is the one field where step 2 (precedent) is not a safe substitute for step 1 (this tour's own confirmed data)** — it's real financial/booking data tied to an actual contracted rate, and a wrong number here is a wrong price a guest could actually pay. So: still check this tour's own record and Notes for a rate that's already been confirmed and just not entered yet — if it's there, propose it directly, sourced. If it's genuinely not on file anywhere for this tour, still make a specific suggestion rather than leaving it blank, but make the precedent-based nature of the guess and its confidence explicit rather than presenting it as equally solid: "No confirmed rate on file for this tour. The same property/room (Mar de Ar Aqueduto, Standard) is used on [other Alentejo tour] at $X Double / $Y Solo — proposing that as a placeholder, but this needs an explicit confirm from the actual contract/quote before it's treated as real, not just a correct-or-not check." If the tour's own `Status` isn't `Confirmed` yet, say that plainly alongside the proposal — a specific number doesn't mean it's safe to publish against.

This is a permanent behavior for this skill, not a one-off judgment call to make fresh each time: check available information first, always land on a specific proposal for every gap, and scale down confidence/caveat the proposal rather than withholding it when the only source is precedent instead of this tour's own confirmed data.

## Publishing

Once every checklist item passes and the webhook issue above is confirmed fixed:

1. **Get the publish key, live, every time** — read it from Airtable's `Automation Config` table (`Key = "Publish API Key"`, the `Value` field holds it). Don't hardcode or reuse a copy from a previous run; if it's ever rotated, a stale copy would just produce 401s.
2. **Call the endpoint:**
   ```
   curl -sS -X POST https://tintotravels.com/api/publish-tour \
     -H "Content-Type: application/json" \
     -H "X-Publish-Key: <value from Automation Config>" \
     -d '{"tourId":"<the Tour record ID>"}'
   ```
3. **Read the response, don't assume success from a lack of an error:**
   - `200 {"success":true,"tourId":"...","slug":"...","url":"https://tintotravels.com/<slug>","chars":<int>}` — this is the only real success case. Sanity-check `chars` is a plausible page size (comfortably above zero, comfortably under 100,000 — the `Page HTML` field's cap) rather than just checking the status code.
   - `400` — missing `tourId`, or the tour has no `Slug` yet. Go back and set one (readiness checklist above) rather than retrying blindly.
   - `401` — the key is missing or wrong. Re-read it from Automation Config; if it still fails, the Cloudflare-side secret may have changed and needs Sabrina/Peter.
   - `500` — a server-side problem (e.g. the Worker's own `PUBLISH_API_KEY` secret unset). Stop and flag this rather than retrying repeatedly — it's not something this skill can fix.
   - **A request that fails partway on an image-heavy tour** (e.g. Damiani, District Pit — many packages/photos) can hit Cloudflare's per-invocation subrequest cap. This is self-healing: `copyToR2` skips images it's already copied, so simply calling the endpoint again converges. Retry up to 2–3 times before treating it as a real failure rather than a transient cap.
4. **Verify the live page, don't just trust the API response — this step is not optional.** `curl` (or fetch) the returned `url` and confirm it's a real 200 with recognizable content (the tour's actual name/branding, and if this is a republish, the specific content that changed), not a fallback, an error page, or stale content. There's a known, documented routing trap (`deploy-runbook.md`, 2026-08-06) where a static file previously committed into the site's `public/` folder for that same slug wins over anything served from Airtable — in that case `/api/publish-tour` reports `success: true` and genuinely writes the new HTML into the tour's `Page HTML` field, but the live page silently keeps showing the old static content, with no error anywhere in the chain. The only way to catch this is exactly this step: actually looking at the live URL, not trusting a `200`/`success` response. If a republish doesn't seem to have changed anything on the live page, this static-file conflict is the first thing to suspect — flag it rather than repeating the publish call, since repeating it will keep reporting success and keep not working.
5. **Set `Publish Status`** on the Tour record to reflect reality — the field's options are `Draft`, `Preview`, `Published`, `Archived`. Use `Published` once step 4 confirms the live page is correct; use `Preview` if this is a deliberate soft-launch not yet meant for real traffic (check with Sabrina/Peter/Nina if unsure which one applies).

**Note for whoever installs this plugin:** this publish step runs as a shell command (`curl`), which is part of the standard Cowork workspace, not a connector — it doesn't need the Airtable or Gmail connector to be scoped any differently, and it doesn't need a device bridge, repo access, or GitHub at all.

## Escalate rather than guess

Any checklist item that's ambiguous (e.g. a price range with no obvious "lowest room" candidate, a Client record that might match but isn't a clean name match) — flag it rather than picking one. Any endpoint response other than a clean `200` with a sane `chars` value and a verified-live URL — flag it rather than declaring the page published. And regardless of how ready the data looks, never skip the webhook-fix confirmation above for a page that hasn't been published before.
