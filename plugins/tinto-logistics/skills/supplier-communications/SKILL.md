---
name: supplier-communications
description: >
  This skill should be used when Tamara asks to "check today's supplier
  emails," "run the supplier communications check," "what suppliers need
  reconfirming today," "draft the hotel/winery/restaurant/transport emails,"
  "follow up on a supplier that hasn't replied," or asks how the supplier
  reconfirmation cycle works, why a particular supplier was or wasn't
  emailed, or what to do with a drafted supplier email. It covers the full
  supplier communications cycle: checking Airtable for what's due, drafting
  into Gmail or flagging for WhatsApp, and the human review/send step.
metadata:
  version: "0.1.0"
---

# Supplier Communications

Guide Tamara through Tinto Travels' supplier reconfirmation and outreach cycle. This skill covers one channel only — **suppliers** (hotels, restaurants, cultural sites, transport, wineries-as-venue) — never guests or wineries-as-client-brand communication: that is Nélia's lane, in a separate plugin, and out of scope here even if asked about. (A winery can appear on both sides of the business — as a supplier providing a tasting/venue, and as a client brand a tour is sold under — this skill only covers the supplier side.)

## Scope check first

Hotels are handled differently from every other supplier category — see "Hotels are the exception" below. If a request is actually about batch-booking a block of hotel rooms ahead of a sale, or converting a sold week into a live booking, that's the `batch-book-hotel-rooms` skill, not this one.

## The mechanism, in outline

Every category books **rolling, single-tour-at-a-time**, except hotels. For each supplier category and tour, there are up to three reconfirmation touchpoints after first contact, each triggered off a different rule:

1. **Room Count Reconciliation** (hotels only) — cancel unsold held rooms before the hotel's free-cancellation deadline. Read the lead time from that specific hotel's row in `Supplier Booking Lead Times` — it varies hotel to hotel, never assume a fixed buffer.
2. **Final Reconfirmation** — 28 days before a tour, confirm final headcount. Applies to Hotel/Winery/Restaurant/Transport, not Tour Guides. Food sensitivities are confirmed as part of this touchpoint too, but only for Hotel/Winery/Restaurant — never Transport.
3. **Season Reconfirmation** — calendar-anchored, not day-count: every **November 3rd** for upcoming Spring tours (Start Date March–July), every **April 1st** for upcoming Fall tours (Start Date September–November). A lighter "still on for these dates" touch, not a final-headcount ask. Applies to Hotel/Winery/Restaurant/Transport.

**Hotel Final Confirmation** is a fourth touchpoint, hotel-only, and different again — not day-count at all: it triggers when a tour hits **70% full**.

When asked to run this check (or picking up a scheduled run):

1. **Read live, don't assume.** Look up the current trigger fields, lead times, and dedup status directly on the production Airtable base's `Suppliers`, `Supplier Booking Lead Times`, and `Bookings (Confirmations)` tables rather than relying on a fixed list of timings written down here — these get revised without notice.
2. **Identify what's due today**, per supplier per tour, against the rules above.
3. **Check the supplier's `Preferred Communication Channel`** (Email / WhatsApp / Other) before drafting — see "Channel handling" below.
4. **Check the supplier's `Language Preference`** and draft in that language — default is the supplier's own local language unless the field overrides it. This is a genuinely different rule from the guest/winery-client side (always American English) — don't apply that rule here.
5. **Draft into Gmail — never send.** Sending is Tamara's deliberate human step.
6. **Report back plainly**: which suppliers got a new draft, for which touchpoint, which ones need manual WhatsApp/Viber copy-over, and anything ambiguous (see Escalation below).

## Hotels are the exception

Hotels don't get a rolling per-tour first-contact row like every other category. Instead, Tamara or Nélia requests a block of rooms ahead of any specific sale (`Hotel Room Inventory` table, `Status` = Available/Sold). When a hold actually sells, a new `Supplier Booking Lead Times` row is created for that Tour × Hotel — without that step the sale sits in `Hotel Room Inventory` but none of the three touchpoints above will ever fire for it. If a sold hotel week seems to be missing from the reconfirmation cycle, check for this first before assuming something else is broken.

## Channel handling

The draft is always written the normal way in Gmail regardless of channel — WhatsApp-preferred and Viber-preferred suppliers still get a Gmail draft, it's just copied over by hand afterward rather than sent from Gmail. **There is no automated WhatsApp send — flag every WhatsApp/Viber-channel draft explicitly as an action item**, the same way the guest-side skill flags missing attachments. "Other" mostly means Viber (several Greek suppliers) or, for the two no-contact-at-all cultural sites (Cathedral of Évora, Museu Berardo Estremoz), "don't reach out — a local host buys tickets in person, day-of, cash." Check the record directly rather than assuming from category or tour — WhatsApp-preferred suppliers span every destination, not just one.

## Escalation (no reply after sending)

7 days after a supplier email is actually sent, with no reply → auto-drafted follow-up. 14 days total, still nothing → stop drafting, flag for a phone/WhatsApp follow-up instead — the email channel is treated as exhausted for that touchpoint.

## What's Tamara's to change, and what isn't

Tamara owns supplier email content end to end — she can edit a drafted email's wording herself, and she can change the underlying template itself, not just one draft's wording. This is the same ownership Nélia has over guest/winery templates; it's the substance of her job, not a system-wide process change. She does not need to ask permission to reword a template or hold off on sending one.

She should **not** change: the trigger timing/lead-time windows, the 70%-full threshold, the Airtable schema, or the scheduler itself. Those are system-wide rules that stay with Peter and Nina. If a change like that seems needed, say so plainly and suggest she raise it with them.

**Holding a shaky booking needs no special mechanism.** Once a row is drafted, it isn't re-drafted or re-escalated the next day just because the underlying condition (e.g. `% Full` ≥ 70%) still holds — a row only drafts once per touchpoint. So if Tamara wants to hold off on confirming a specific hotel booking, simply not sending the draft already achieves that; nothing automatic nags or re-fires as a result. The 7-/14-day escalation logic only starts once a row is actually marked sent.

## Escalate, don't fix, when you see

See `references/conventions-and-escalation.md` for the full list, mirroring the guest-side skill's escalation pattern.
