# Conventions and Escalation Reference

Detail supporting the `daily-guest-communications` skill. Load this when actually drafting or reviewing an email, or when deciding whether something needs to be escalated.

## Currency

Guest-facing amounts (bookings, deposits, balances) are always **USD**, regardless of the tour's destination. Never convert to EUR or any other currency without being explicitly asked to.

## Language

Both of Nélia's channels — **guest** communication and **winery** communication — are always **American English**, with no exceptions by destination. This holds even for tours to non-English-speaking regions (Peloponnese, Loire Valley, Northern Adriatic & Slovenia, etc.).

This is a genuinely different rule from supplier language, which varies per supplier (Tamara's lane — see the Logistics plugin's `Language Preference` field). Don't apply supplier-language logic here; it doesn't exist on this side of the business.

**The `Regional Sign-Off` field is not an exception to the English rule** — it's a single deliberate destination-language flourish inside an otherwise fully English email (e.g. ending with "Obrigada" for a Portugal tour). If a draft looks like it has one foreign word in an English email, that's correct, not a mistake to fix:

| Destination | Regional Sign-Off |
|---|---|
| Alentejo, Porto & Douro (Portugal) | Obrigada |
| Southern Tuscany & Umbria, Puglia (Italy) | Grazie |
| Castille & León (Spain) | Gracias |
| Peloponnese (Greece) | Efcharistó |
| Northern Adriatic & Slovenia (Italy + Slovenia) | Grazie / Hvala |
| Austria | Danke |
| Loire Valley (France) | Merci |

If a tour's destination isn't listed here, check the `Regional Sign-Off` field on the Tour record directly rather than guessing.

## Attachments to check for, by touchpoint

The Gmail draft never contains these — they're delivered to Nélia separately and must be attached by hand before sending:

- **Welcome** — Pre-Trip Guide PDF
- **6-Weeks-Out** — Pre-Trip Guide PDF (again)
- Any touchpoint carrying trip logistics — itinerary PDF (build this with the `guest-itinerary-pdf` skill, elsewhere in this plugin, if it doesn't already exist for the tour)
- **Approval Request** (winery-facing) — the merged post-trip email draft rides along with this one

If a touchpoint isn't listed here, it likely doesn't need an attachment — but when in doubt, check rather than assume either way.

## Escalate, don't fix, when you see:

- A drafted email addressed to what looks like the wrong guest, or referencing a tour/winery that doesn't match the reservation.
- What looks like a duplicate draft for a touchpoint that should already be marked sent or drafted (the dedup field should have prevented this — if it didn't, something's inconsistent upstream).
- A reservation or tour record that's missing information a template needs (e.g. no linked winery, no dates), producing a draft with blanks or clearly wrong content.
- Any situation where fixing it would mean editing the trigger timing, the dedup logic, or the Airtable schema — that's Peter/Nina's call, not something to patch through this skill.

In all of these cases: stop, don't send or "fix" the draft, and tell Nélia (or whoever's running the check) plainly what looked wrong and why — then flag it to Sabrina, Peter, or Nina. Guessing at a workaround risks a guest or winery getting a genuinely wrong email, which is worse than a short delay while someone checks it.
