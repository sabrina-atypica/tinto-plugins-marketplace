# Conventions and Escalation Reference

Detail supporting the `supplier-communications` skill. Load this when actually drafting or reviewing a supplier email, or when deciding whether something needs to be escalated.

## Language

A per-supplier `Language Preference` field on the `Suppliers` table. Default assumption is the supplier's own local language unless the field overrides it. In practice most suppliers on file are set to English regardless of home country, including every Greek supplier (a deliberate Greek-always-English policy) — the current exceptions are Pretto and Fabio Gervasoni (Tuscany), both Italian. There's no French supplier on file yet; when one shows up, treat it case-by-case rather than assuming English works — check with Tamara.

This is a genuinely different rule from the guest/winery-client side (always American English, no exceptions) — don't apply that rule here, and don't assume a supplier email should be in English just because the destination's guest emails are.

## Currency

Supplier-facing terms (room rates, deposits, cancellation penalties) follow whatever currency that supplier actually quotes in — check the supplier's own record or prior correspondence rather than assuming USD, which is a guest-facing convention only.

## Channel quick reference

| Channel | What to do |
|---|---|
| Email | Draft and send from Gmail normally. |
| WhatsApp | Draft in Gmail as usual, then flag clearly that it needs manual copy-over to WhatsApp before it reaches the supplier — nothing sends automatically. |
| Other (Viber) | Same as WhatsApp — draft in Gmail, flag for manual copy-over to Viber. |
| Other (no-contact site) | Do not draft anything. Cathedral of Évora and Museu Berardo Estremoz are cash-on-arrival, no outreach at all. |

## Escalate, don't fix, when you see

- A drafted email addressed to what looks like the wrong supplier, or referencing a tour that doesn't match the booking.
- What looks like a duplicate draft for a touchpoint that should already show as sent (the dedup/status field should have prevented this — if it didn't, something's inconsistent upstream, flag rather than patch).
- A supplier record missing a field a template needs (no `Language Preference`, no `Preferred Communication Channel`, a blank `Hotel Cancellation Policy` for a hotel that's about to hit Room Count Reconciliation) — producing a draft with blanks or a guess baked in.
- A hotel's sold week that isn't triggering any reconfirmation touchpoints — check first whether a `Supplier Booking Lead Times` row was actually created for that Tour × Hotel at the point of sale (see "Hotels are the exception" in the main skill) before assuming the scheduler is broken.
- Any situation where fixing it would mean editing lead-time windows, the 70%-full threshold, or the Airtable schema — that's Peter/Nina's call.

In all of these cases: stop, don't send or "fix" the draft, and tell Tamara (or whoever's running the check) plainly what looked wrong and why — then flag it to Sabrina, Peter, or Nina. Guessing at a workaround risks a supplier getting a genuinely wrong email, or a real cancellation deadline being missed, which is worse than a short delay while someone checks it.
