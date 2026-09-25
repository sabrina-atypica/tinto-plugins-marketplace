# Tinto Logistics

Onboards Tamara (Tinto Travels' supplier communication and logistics owner) onto the automated daily supplier-booking check and drafting cycle.

## Overview

This plugin covers exactly one thing: Tamara's daily supplier-booking cycle — checking Airtable for what's due each day, drafting supplier emails correctly, reviewing/sending them. It deliberately does **not** cover guest or winery communications (Nélia's separate `tinto-guest-communication` plugin, installed the same restricted way — only Nélia and Tamara install their respective daily-cycle plugins), system-wide automation rules (trigger timing, dedup logic, schema — Peter/Nina's), or Sales/BD. **Split 2026-09-15:** `confirm-supplier-pairings`, `batch-book-hotel-rooms`, `rooming-lists`, and `bus-company-itineraries` used to be bundled here — they're now their own standalone plugins, installable by anyone on the team who needs them, not just Tamara. Only this plugin and `tinto-guest-communication` are restricted to one specific person's daily workflow; every other Tinto plugin in the marketplace is open to the whole team.

Three pieces of feedback from Tamara directly shaped the rebuild of the one skill still in this plugin:

1. **Every supplier email drafts in English first**, for her own review, regardless of that supplier's `Language Preference` field. Translation, if the final send needs one, is a separate step done only after she's reviewed and is happy with the English content — never before.
2. **At the end of every run, Tamara gets pinged directly in her own Claude chat** with a plain overview of which emails were due today and confirmation the drafts are sitting in Gmail — not a silent Airtable update she has to go check for herself.
3. **If a supplier email belongs to an existing Gmail thread, the draft goes into that thread** — a proper in-thread reply — never as a new, disconnected email.

## Components

| Skill | Purpose |
|---|---|
| `daily-supplier-communications` | The core process: how the daily scheduler decides what's due, drafting into Gmail (English-first, thread-aware), reviewing/sending, the end-of-run ping, and the line between what Tamara can change herself and what stays with Peter/Nina. Includes the twice-yearly Season Reconfirmation sweep (Nov 3 / Apr 1) — same daily scheduled task, no separate one needed. Will not draft to a pairing the `confirm-supplier-pairings` plugin's skill hasn't cleared yet — see its own "Gate" section. |

No agents or hooks — enforcement of what Tamara can and can't change is already handled by her Airtable seat permissions (add/delete data, no schema changes), not by anything in this plugin.

## Setup

Requires the org's existing **Airtable** and **Gmail** connectors to be enabled for Tamara's Cowork account. No GitHub or Cloudflare access of any kind is needed.

**New tours need linking to their suppliers before this plugin can contact anyone for them.** When Peter or Nélia sell a tour, it has no supplier rows at all. Every daily run lists these tours at the top of the ping as "sold, not yet linked," until Tamara links them with the `link-tour-suppliers` skill in the separate `tinto-confirm-supplier-pairings` plugin (install it too). As of 2026-09-25 the existing tours' rows were synced to `Bookings (Confirmations)` and mostly carry a confirmation marker already; the few rows still unconfirmed are asked about in the ping when they fall due.

**Correcting a supplier at drafting time.** If Tamara says a supplier is wrong for a tour when a draft is about to go out, the skill updates both `Bookings (Confirmations)` and `Supplier Booking Lead Times` for that tour and redrafts to the right supplier, rather than only fixing the email.

**The daily check needs a scheduled task to actually run automatically, and it needs to be set up a specific way for the "ping" feedback item to actually work:**

- Create it with the Claude Code Remote scheduled-task tools (`create_trigger`), never a local/in-process cron — a locally-scheduled task is lost the moment the session ends and would silently never fire.
- Run it daily (a morning time that suits Tamara's own working hours is fine — match whatever Peter/Nina's existing Module 1 scheduler uses unless Tamara asks for something different).
- **Turn on push notifications for this scheduled task** (`notifications: {push: true}`, email optional per her preference). Each firing starts a fresh session, and that new session **is** the ping — Tamara sees it appear as a new chat in her Claude, and the push notification is what actually pulls her attention to it rather than leaving her to notice it on her own.
- The scheduled task's prompt should simply ask Claude to run the `daily-supplier-communications` skill. Everything else — what to check, how to draft, what to say at the end — is in that skill.

This is a one-time setup step outside the plugin itself, same as Nélia's daily check needed. **No second scheduled task is needed for the November 3rd / April 1st Season Reconfirmation sweep** — it's a calendar check the same daily task runs every time it fires, not a separate trigger. Season Reconfirmation and the day-count touchpoints (First Contact, Final Reconfirmation, Room Count Reconciliation, Hotel Final Confirmation) are both covered by `daily-supplier-communications` end to end — see that skill's own "Season Reconfirmation" section for exactly how the twice-yearly check works and what it looks for.

One caveat worth knowing before the first Nov 3 / Apr 1 sweep actually fires: the documented design has Season Reconfirmation covering Hotel/Winery/Restaurant/Transport, same as Final Reconfirmation — but production's `Supplier Booking Lead Times` table currently has no Final Reconfirmation rows for Hotel at all (only Winery/Restaurant/Transport got them in the 2026-09-09 bulk wire). Hotels still have other rows the Season Reconfirmation sweep can work from, so this doesn't block it — but it's worth Sabrina/Peter/Nina confirming whether that's intentional (hotels already get three other touchpoints) or a real gap in production data.

## Usage

Ask Claude things like:
- "Check today's supplier emails" / "What's due today?" — runs the daily check and drafting.
- "Why didn't [supplier] get an email?" / "Is this a duplicate?" — troubleshoots using the same skill.

## Also worth Tamara's attention, not part of this plugin's daily mechanism

Production's `Bookings (Confirmations)` table was just populated (2026-09-14) with 458 rows extracted directly from the real 2027 guest itinerary PDFs — every row is tagged either a clean match or an unconfirmed candidate needing her (or Nélia's) sign-off before it should be treated as an actually-confirmed booking. A separate flagged list of 22 real suppliers/venues that came up in those itineraries but have no Suppliers record yet lives at `claude/itinerary-new-suppliers-flagged.md` in the project. Worth a pass through both once this plugin is installed — neither needs this plugin to review, they're just sitting there waiting on her.

## Not in this plugin

Guest and winery communications — `tinto-guest-communication` (Nélia's equivalent restricted plugin). Linking a newly sold tour to its suppliers — `tinto-confirm-supplier-pairings`. Batch-booking hotel rooms — `tinto-batch-book-hotel-rooms`. Rooming lists — `tinto-rooming-lists`. Bus company itineraries — `tinto-bus-company-itineraries`. Booking-page publishing — `tinto-booking-pages`. All of these are separate installs, open to anyone on the team who needs them, not restricted the way this plugin and `tinto-guest-communication` are. Sales/BD, and anything that would change trigger rules, lead-time windows, or Airtable schema itself, stay Peter/Nina's, same boundary as the rest of Module 1.
