---
name: bus-company-itineraries
description: >
  This skill should be used when Tamara asks to "create the bus itinerary
  for [tour]," "put together the transport schedule for [tour]," "send the
  driver our day-by-day pickup schedule," or needs the plain, day-by-day
  pickup/dropoff document a tour's Transport supplier (a bus or driver
  company) actually works from.
metadata:
  version: "0.4.0"
---

# Bus Company Itineraries

Produce the day-by-day driver schedule Tamara sends straight to a tour's Transport supplier. This is a **plain internal working document**, not guest-facing marketing copy and not one of Tinto's branded PDFs — see "Format and layout" below for why. **The format is not one universal template — it varies by destination** (language, header shape, level of detail). Always look up the tour's own destination convention in `references/destination-formats.md` before composing anything; don't default to whichever destination you've seen most recently.

## Onboarding note added 2026-09-24 — first-use coverage check-in

The very first time this skill runs for Tamara after the plugin is handed over to her (she's just installed it, hasn't used it yet, or opens with something like "what does this do" or "I just added this plugin"), lead with a short coverage check-in before doing anything else — including before working on any specific tour she may have also asked about in the same message:

- Tell her which destinations already have a bus-itinerary format template on file, and that any future itinerary for one of these destinations will be built from that template: Alentejo, Loire Valley, Northern Adriatic & Slovenia, Southern Tuscany & Umbria, Castille & León.
- Tell her which of Tinto's other destinations (per the full `Location` choice list on `Tours`, as of 2026-09-24) don't have a template yet: Porto & Douro, Coastal Tuscany, Peloponnese, Puglia, Douro, Vinho Verde, Austria. If `Tours.Location` has grown new choices since, re-check the live list rather than trusting this one as permanent.
- Ask if she has a template ready for one of those missing destinations right now, and if so, to just drop it in the chat — it becomes a new section in `references/destination-formats.md`, the same way the existing five were built.
- If not, tell her that's fine: the next time an actual bus itinerary is needed for one of those destinations, prompt her for a template for that specific tour then, rather than asking her to produce one for every missing destination up front (see "Escalate rather than guess").

Do this once per onboarding, not on every request afterward — once she's answered (with a template or "not yet"), move straight to whatever she actually asked for.

## Rebuilt 2026-09-16

The original version of this skill assumed `Bookings (Confirmations)` rows tagged `Supplier.Type = Transport` were the whole story, and that the output should be a branded PDF like the Pre/Post-Tour Planning Guide. Neither held up against a real example Sabrina provided (`BUS_2026_06_08_Linganore_alentejo.pdf`):

- The real document is unbranded plain text — a bold header line, underlined day headers, and time-stamped entries. No logo, no color palette, no Cormorant Garamond/Inter treatment.
- It needs a driver's whole day, not just formally-tagged Transport rows: pickups, drop-offs, winery/restaurant/cultural-site visits, comfort stops, luggage handling, and the return-to-hotel leg all appear as entries a bus has to physically execute.
- It needs exact clock times ("09:45", "12:15"), which `Bookings (Confirmations)` didn't previously store — only a coarse `Time Slot` bucket (Morning/Lunch/Afternoon/Evening).
- It has a header with two named on-the-ground contacts and phone numbers (e.g. "Andreia: 961 792 740", "Nina: 91 639 1989") that don't match any Suppliers record — checked against the actual Transport supplier on file for Alentejo (RSI, contact André Lopes) and it's a different person. These are Tinto's own people for that tour run, not a supplier's.

Four fields were added to `Bookings (Confirmations)` and one to `Tours` to close these gaps (see "Where the data comes from"). **Historic tours won't have this data populated yet** — this is new schema, not backfilled — so expect to escalate to Tamara for older/sparse tours until she's had a chance to fill it in going forward.

## Extended 2026-09-17 — the format varies by destination

Tamara separately provided five more real examples, one each for Alentejo, Loire Valley, Northern Adriatic & Slovenia, Southern Tuscany & Umbria, and Castille & León. Comparing them showed the Alentejo/Portuguese layout above is just one of several real conventions, not a universal template: Loire is French with drive distances between stops, Slovenia is English with a formal title-block header and bullets, Tuscany is English with a plainer dash-separated layout and no pax count, and Castille & León is Spanish with numbered stops. `references/destination-formats.md` captures all five conventions in detail — read the section matching the tour's `Location` before composing (see "Which destination convention to use").

None of those five source documents correspond to a tour currently in production Airtable (all are 2026-dated; every tour in the base as of 2026-09-17 is 2027-dated) — they're format references only, not data to reprint for a real request.

## Where the data comes from

**Primary source of truth: the production Airtable base.** Pull, for the requested tour:

From `Tours`:
- `Start Date`, `End Date` — for the header date range and to enumerate every day of the tour.
- `Guests Booked` — total pax for the header.
- `On-Tour Contacts` — the "Name: Phone" lines for the header. If blank, ask Tamara for the on-the-ground contact(s) rather than omitting the header line or inventing a name.

From `Bookings (Confirmations)`, filtered to this tour: **pull every row, in `Date` then `Time` order — do not filter to Supplier.Type = Transport.** The driver needs the whole day's sequence (a winery visit or a lunch stop is still a stop the bus has to make), not just rows formally tagged as a transport booking.

- `Time` — the exact clock time for the entry. If a row for a day that clearly needs one (e.g. a morning departure) has no `Time` set, flag it to Tamara rather than guessing a plausible-looking time.
- `Stop Type` — Pickup / Drop-off / Transfer / Waypoint / Sightseeing / Meal / Comfort Stop / Luggage Handling / Return to Accommodation / Other. Use this to decide the phrasing convention (see "Composing each line"), not to decide whether to include the row — every row for the tour is included.
- `Activity Description`, `Supplier` (linked record's name), `Location / Address Note` — together, what and where. A row can have a Supplier, a free-text Location / Address Note, or both (e.g. a WC stop near Vendas Novas has only a location note, no Supplier).
- `Logistics Note` — driver-facing operational detail: luggage handling, a wait duration, anything that changes what the driver actually does. Fold this into the line naturally rather than appending it as a separate clause every time.
- `Notes` — this is extraction/confirmation provenance (how confident Claude or Tamara is that this row is right), not driver-facing content. Don't put it in the document; do read it, since a row still marked as an unconfirmed guess is a reason to double-check with Tamara before sending the schedule to a real driver.

If a day in the tour's date range has no rows at all, that's a gap — ask Tamara whether the bus genuinely isn't needed that day (a rest day, a day where guests explore independently) or whether the day's schedule just hasn't been entered yet. Don't silently produce a day with no entries and don't invent one either.

`Standard Itineraries` can still be checked as a rough baseline if a tour looks unusually sparse, but every row there is unconfirmed by design (`Confidence` field) and several destinations were reconstructed rather than pulled from real bookings — treat it as a hint about what's plausible, never as data to print on a driver's schedule.

## Which destination convention to use

Before composing anything, get the tour's `Location` from `Tours` and read the matching section of `references/destination-formats.md`. That file is the actual source of truth for language, header shape, day-header format, entry style (plain lines vs. bullets vs. numbered stops), whether drive distances are shown, and date/weekday formatting conventions for each destination seen so far (Alentejo, Loire Valley, Northern Adriatic & Slovenia, Southern Tuscany & Umbria, Castille & León). Don't reuse another destination's conventions by analogy, and don't invent one — if the tour's `Location` has no section there yet, ask Tamara for a real example first, the same way the existing five got built.

## Composing each line

The source data is structured (Activity Description, Supplier, Location / Address Note, Logistics Note); the output line is a short, natural sentence in the plain operational style of that destination's own convention — not a mechanical concatenation of the fields with labels, and not a translation of another destination's phrasing into a different language. General principles that hold across every destination:

- Lead with the time, then the action, in whatever phrasing convention that destination's reference uses (e.g. Alentejo's "saída para X" / "voltar para o hotel"; Slovenia's "Departure for X").
- Fold luggage handling in naturally, in the destination's own shorthand (Alentejo: "+ MALAS", "COM MALAS"; Castille & León: "COM TODA SU EQUIPAJE"; Loire/Slovenia: "with luggage" / "+ bagages" phrased in prose).
- A wait or comfort stop gets a short clause in the same style as the rest of that destination's document (e.g. Alentejo's "parando por 40 minutos em X" / "fazer uma paragem para WC").
- When a Supplier and a separate Location / Address Note both describe the same stop, don't repeat the place name twice — pick whichever reads more naturally as the destination.
- When two rows share the same exact `Time`, render them as consecutive lines without repeating the time (see the Alentejo 12:15 example in the reference file).
- Match the destination's own language exactly (Portuguese for Alentejo, French for Loire Valley, English for Slovenia and Tuscany, Spanish for Castille & León) — this is an operational document for Tinto's own on-the-ground team and a local transport supplier, distinct from the English guest-facing materials this marketplace otherwise produces (and distinct from Sabrina's own English-language preference for what Claude says to her — that governs conversation and other deliverables, not a destination document's actual content, which follows the local convention).
- Cross-check any pax count that appears inside a note against `Tours.Guests Booked` rather than trusting the note in isolation — see the general rule at the top of `destination-formats.md` about the Alentejo header/body mismatch.

## Format and layout

Reproduce that destination's own header shape, day-header format, and entry style exactly, per `references/destination-formats.md` — this is a plain internal working document, not one of Tinto's branded guest-facing PDFs, so **do not** apply `tinto-brand.md`'s palette, typefaces, or logo here for any destination. No color, no logo, no decorative elements, regardless of language.

Build the actual file with the pdf skill (read its SKILL.md before generating) — the formatting here is plain text layout (bold/underline/bullets as that destination's convention calls for, a simple header block), not a styled document, so keep the pdf skill's own defaults minimal rather than reaching for a template.

## Escalate rather than guess

If a day is missing entries entirely, a time is missing where one is clearly needed, the tour's `On-Tour Contacts` is blank, a row's `Notes` still marks it as an unconfirmed guess, or the tour's `Location` has no convention on file in `destination-formats.md` yet, say so plainly and ask Tamara rather than inventing a plausible-looking schedule or format — a driver working from a wrong or incomplete schedule is a real operational failure, not just an inconvenience.
