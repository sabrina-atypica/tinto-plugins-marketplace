---
name: booking-page-publishing
description: >
  This skill should be used when Nélia or Tamara asks to "check if [tour]'s
  booking page is ready," "publish [tour]'s page," "why isn't [tour]
  bookable yet," or needs to audit and publish a tour's public sign-up/
  booking page.
metadata:
  version: "0.1.0"
---

# Booking Page Readiness and Publishing

Audit a tour's Airtable data against what its public booking page actually needs, then help get it live. **Read the two warnings below before doing anything else in this skill — they materially change what "safe to publish" means right now, as of 2026-09-14.**

## ⚠ Known live issue: guest payments may not be reaching Airtable

An audit on 2026-09-14 found that the pending-write step from a booking page's checkout flow into Airtable's `Reservations` table is **not currently working in production**, confirmed by the complete absence of any `TT-`prefixed booking reference in `Reservations`, including for a real completed test payment. **If this is still unresolved, a real guest who completes a payment on a live booking page today would have no Airtable record of it at all** — no reservation, no confirmation trail, nothing for the rest of the automation to act on.

**Before publishing any tour's page for the first time, or promoting a preview to live, confirm directly with Sabrina, Peter, or Nina that this has been fixed and verified with a real test transaction.** Do not treat the checklist below as "safe to publish" on its own — a page can pass every readiness item and still silently lose real bookings if this is still broken. If you can't get a clear confirmation that it's fixed, say so plainly and hold off on the publish step, even if asked to proceed — flag it rather than guess.

## ⚠ Publishing itself needs repo/deploy access, which most Cowork sessions don't have

Per this plugin's own original scoping (2026-09-08), the mechanism that actually renders a tour's live page runs page-generation code from the booking-page repository (`buildPageData`/`renderTourPage`) and deploys it (Cloudflare) — a plain Cowork session with only the Airtable/Gmail connectors cannot do this step. If this session doesn't have a device bridge with repo access (check for `mcp__remote-devices__*` tools and a connected folder with the site repo, e.g. Sabrina's `TintoTravels` folder), **do the readiness audit below, report the results, and hand the actual publish step to whoever does have that access** — don't claim to have published a page you couldn't actually deploy.

## Readiness checklist

Pull the tour's record from the production Airtable base and check each of these directly — don't assume from a spreadsheet or prior conversation, the live base is the source of truth:

| Field | Requirement | Failure mode if missing |
|---|---|---|
| `Slug` | Set, unique | Page has no URL |
| `Price per Person` | Set (use the *lowest bookable room's* price if the tour has a price range — this is the existing convention, not a special case) | Renders literal "$NaN" on the hero |
| `Overview`, `Tour Highlights`, `What's Included` / `What's Not Included` | Populated, guest-facing copy | Blank sections on the page |
| Hero / Gallery photos | At least one, on the Tour or inherited from its `Destinations` record | Degrades gracefully to a standard gradient hero if blank — not blocking, but check before promoting externally |
| Linked `Client` (not the plain-text `Client / Affiliation` field) | Populated | Zero client branding (logo/accent color) even if the plain-text field looks filled in — these are two different fields, check the linked one specifically |
| Linked `Itinerary Days` | Populated, matches the tour's actual confirmed schedule | Guests see no day-by-day content, or content that doesn't match what they booked |
| Linked `Packages` | At least one row, `Live Status` = Bookable (not Pre-registration or another status) | Nothing for checkout to sell — page renders but guests can't book |

**Also check, not blocking but worth knowing:** whether the tour's `Packages` rows share a hotel room block across Double/Solo occupancy types — if so, their `Capacity` fields may double-count the same physical rooms (a known data-model issue as of 2026-09-10, see the `rooming-lists` skill in the Logistics plugin for detail). This doesn't block publishing, but means any "X rooms left" messaging shouldn't be trusted without checking with Sabrina first.

## Publishing

Once every checklist item passes and the webhook issue above is confirmed fixed:

1. If this session has repo/deploy access, follow the existing deploy process (`deploy-runbook.md` if available in this Cowork Project) to generate and deploy the page, then set the tour's `Publish Status` field accordingly.
2. If this session does not have that access, report the readiness results clearly (pass/fail per checklist item) and say plainly that publishing itself needs to be done by someone with repo access — don't attempt a workaround.

## Escalate rather than guess

Any checklist item that's ambiguous (e.g. a price range with no obvious "lowest room" candidate, a Client record that might match but isn't a clean name match) — flag it rather than picking one. And regardless of how ready the data looks, never skip the webhook-fix confirmation above for a page that hasn't been published before.
