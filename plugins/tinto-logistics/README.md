# Tinto Logistics

Onboards Tamara (Tinto Travels' supplier communication and logistics owner) onto the automated supplier-booking process.

## Overview

This plugin covers Tamara's day-to-day operation of the Module 1 supplier-booking cycle: checking Airtable for what's due each day, drafting supplier emails correctly, reviewing/sending them, and batch-booking hotel rooms. It also still bundles `rooming-lists` and `bus-company-itineraries` in their original placeholder form (see the note below) — Sabrina has confirmed both are being redesigned as their own separate plugins rather than staying part of this one, but neither redesign has happened yet, so this is still the only implementation of either that exists anywhere. It deliberately does **not** cover guest or winery communications (Nélia's separate `tinto-guest-communication` plugin), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD.

Three pieces of feedback from Tamara directly shaped the supplier-communications rebuild below, all incorporated:

1. **Every supplier email drafts in English first**, for her own review, regardless of that supplier's `Language Preference` field. Translation, if the final send needs one, is a separate step done only after she's reviewed and is happy with the English content — never before.
2. **At the end of every run, Tamara gets pinged directly in her own Claude chat** with a plain overview of which emails were due today and confirmation the drafts are sitting in Gmail — not a silent Airtable update she has to go check for herself.
3. **If a supplier email belongs to an existing Gmail thread, the draft goes into that thread** — a proper in-thread reply — never as a new, disconnected email.

## Components

| Skill | Purpose |
|---|---|
| `confirm-supplier-pairings` | **Run this first, before the first daily check.** Three passes per destination: who the supplier actually is (region-matched pairings checked against real itinerary evidence), what's already happened with them (a Gmail scan for existing correspondence, so real-world contact history isn't lost), and whether the assigned lead time looks right (Tamara can flag one for Peter/Nina, never change it herself). |
| `daily-supplier-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail (English-first, thread-aware), reviewing/sending, the end-of-run ping, and the line between what Tamara can change herself and what stays with Peter/Nina. Includes the twice-yearly Season Reconfirmation sweep (Nov 3 / Apr 1) — same daily scheduled task, no separate one needed. Will not draft to a pairing `confirm-supplier-pairings` hasn't cleared yet — see that skill's "Gate" section. |
| `batch-book-hotel-rooms` | Requesting a new block of hotel rooms for an upcoming season, and converting a sold block into the rest of the automation once a tour books it. |
| `rooming-lists` | **Original placeholder design, not yet rebuilt for Tamara's actual feedback.** Building a per-tour rooms-and-guests spreadsheet from confirmed Reservations — deliberately does not compute "rooms remaining" given the historical Packages-table capacity double-counting issue (now fixed via `Room Blocks`, see `logistics-reference.md`). Sabrina has said this will include food sensitivity/allergy data and ship as its own separate plugin eventually — treat this version as provisional. |
| `bus-company-itineraries` | **Original placeholder design, not yet rebuilt.** Building a branded PDF day-by-day pickup/dropoff schedule for a tour's Transport supplier. Also planned as a separate plugin eventually — treat this version as provisional. |

No agents or hooks — enforcement of what Tamara can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors to be enabled for Tamara's Cowork account. `rooming-lists` also needs the xlsx skill available in the session; `bus-company-itineraries` needs the pdf skill. Both are bundled with Cowork by default. No GitHub or Cloudflare access of any kind is needed for anything in this plugin.

**Before setting up the daily check, run `confirm-supplier-pairings` once, destination by destination.** Most of production's `Supplier Booking Lead Times` table was built by matching a tour's region against a supplier's region, not from a confirmed itinerary — `daily-supplier-communications` will not draft to a pairing that hasn't been through this review (see its "Gate" section), so skipping this step just means the first several real runs report a growing list of "awaiting pairing confirmation" items instead of actual drafts. As of 2026-09-14, only a handful of pairings are due soon enough to matter for the very first run, so this doesn't have to be finished all at once — but it's the sensible first thing to do after installing, before turning on the schedule below.

This same pass is also how real-world history that predates this system gets captured, rather than lost: for every pairing Tamara confirms, the skill checks Gmail for any correspondence that already exists with that supplier and asks her what it means for this tour, so a relationship that's already underway doesn't get treated as untouched. It's also her one chance to flag — not change — a lead time that looks wrong for a specific supplier; the actual policy stays Peter/Nina's call.

**The daily check needs a scheduled task to actually run automatically, and it needs to be set up a specific way for the "ping" feedback item to actually work:**

- Create it with the Claude Code Remote scheduled-task tools (`create_trigger`), never a local/in-process cron — a locally-scheduled task is lost the moment the session ends and would silently never fire.
- Run it daily (a morning time that suits Tamara's own working hours is fine — match whatever Peter/Nina's existing Module 1 scheduler uses unless Tamara asks for something different).
- **Turn on push notifications for this scheduled task** (`notifications: {push: true}`, email optional per her preference). Each firing starts a fresh session, and that new session **is** the ping — Tamara sees it appear as a new chat in her Claude, and the push notification is what actually pulls her attention to it rather than leaving her to notice it on her own.
- The scheduled task's prompt should simply ask Claude to run the `daily-supplier-communications` skill. Everything else — what to check, how to draft, what to say at the end — is in that skill.

This is a one-time setup step outside the plugin itself, same as Nélia's daily check needed. **No second scheduled task is needed for the November 3rd / April 1st Season Reconfirmation sweep** — it's a calendar check the same daily task runs every time it fires, not a separate trigger. Season Reconfirmation and the day-count touchpoints (First Contact, Final Reconfirmation, Room Count Reconciliation, Hotel Final Confirmation) are both covered by `daily-supplier-communications` end to end — see that skill's own "Season Reconfirmation" section for exactly how the twice-yearly check works and what it looks for.

One caveat worth knowing before the first Nov 3 / Apr 1 sweep actually fires: the documented design has Season Reconfirmation covering Hotel/Winery/Restaurant/Transport, same as Final Reconfirmation — but production's `Supplier Booking Lead Times` table currently has no Final Reconfirmation rows for Hotel at all (only Winery/Restaurant/Transport got them in the 2026-09-09 bulk wire). Hotels still have other rows the Season Reconfirmation sweep can work from, so this doesn't block it — but it's worth Sabrina/Peter/Nina confirming whether that's intentional (hotels already get three other touchpoints) or a real gap in production data.

## Usage

Ask Claude things like:
- "Audit the supplier pairings for Alentejo" / "Go through the pairings destination by destination" — runs the pairing-confirmation review (start here).
- "Check today's supplier emails" / "What's due today?" — runs the daily check and drafting.
- "Why didn't [supplier] get an email?" / "Is this a duplicate?" — troubleshoots using the same skill.
- "Request a block of rooms from [hotel] for [season]" / "[Tour] just sold, book the rooms" — batch-books or converts a hotel room hold.
- "Create the rooming list for [tour]" — builds the rooms-and-guests spreadsheet (placeholder version, see above).
- "Put together the bus itinerary for [tour]" — builds the transport company's day-by-day schedule (placeholder version, see above).

## Also worth Tamara's attention, not part of this plugin's daily mechanism

Production's `Bookings (Confirmations)` table was just populated (2026-09-14) with 458 rows extracted directly from the real 2027 guest itinerary PDFs — every row is tagged either a clean match or an unconfirmed candidate needing her (or Nélia's) sign-off before it should be treated as an actually-confirmed booking. A separate flagged list of 22 real suppliers/venues that came up in those itineraries but have no Suppliers record yet lives at `claude/itinerary-new-suppliers-flagged.md` in the project. Worth a pass through both once this plugin is installed — neither needs this plugin to review, they're just sitting there waiting on her.

## Not yet in this plugin

Anything Sales/BD (landing a new supplier relationship, negotiating terms) and anything that would change the trigger rules, lead-time windows, or Airtable schema itself — those stay Peter/Nina's, same boundary as the rest of Module 1. `rooming-lists` and `bus-company-itineraries` are bundled here only because they haven't been designed as their own separate plugins yet (see the Components table) — don't assume either has had the same rebuild/verification pass the other three skills just got.
