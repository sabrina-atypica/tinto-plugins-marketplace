---
name: bus-company-itineraries
description: >
  This skill should be used when Tamara asks to "create the bus itinerary
  for [tour]," "put together the transport schedule for [tour]," "send the
  driver our day-by-day pickup schedule," or needs the plain, day-by-day
  pickup/dropoff document a tour's Transport supplier (a bus or driver
  company) actually works from.
metadata:
  version: "0.2.0"
---

# Bus Company Itineraries

Produce the day-by-day driver schedule Tamara sends straight to a tour's Transport supplier. This is a **plain internal working document**, not guest-facing marketing copy and not one of Tinto's branded PDFs — see "Format and layout" below for why, and match the reference example exactly rather than dressing it up.

## Rebuilt 2026-09-16

The original version of this skill assumed `Bookings (Confirmations)` rows tagged `Supplier.Type = Transport` were the whole story, and that the output should be a branded PDF like the Pre/Post-Tour Planning Guide. Neither held up against a real example Sabrina provided (`BUS_2026_06_08_Linganore_alentejo.pdf`):

- The real document is unbranded plain text — a bold header line, underlined day headers, and time-stamped entries. No logo, no color palette, no Cormorant Garamond/Inter treatment.
- It needs a driver's whole day, not just formally-tagged Transport rows: pickups, drop-offs, winery/restaurant/cultural-site visits, comfort stops, luggage handling, and the return-to-hotel leg all appear as entries a bus has to physically execute.
- It needs exact clock times ("09:45", "12:15"), which `Bookings (Confirmations)` didn't previously store — only a coarse `Time Slot` bucket (Morning/Lunch/Afternoon/Evening).
- It has a header with two named on-the-ground contacts and phone numbers (e.g. "Andreia: 961 792 740", "Nina: 91 639 1989") that don't match any Suppliers record — checked against the actual Transport supplier on file for Alentejo (RSI, contact André Lopes) and it's a different person. These are Tinto's own people for that tour run, not a supplier's.

Four fields were added to `Bookings (Confirmations)` and one to `Tours` to close these gaps (see "Where the data comes from"). **Historic tours won't have this data populated yet** — this is new schema, not backfilled — so expect to escalate to Tamara for older/sparse tours until she's had a chance to fill it in going forward.

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

## Composing each line

The source data is structured (Activity Description, Supplier, Location / Address Note, Logistics Note); the output line is a short, natural sentence in the plain operational style of the reference document — not a mechanical concatenation of the fields with labels. Match its conventions:

- Lead with the time, then the action: "saída para X", "saída do hotel para X", "pick-up em X", "voltar para o hotel".
- Fold luggage handling in naturally and keep it shouting-case where the reference does: "(24 pessoas + MALAS)", "COM MALAS".
- A wait or stop gets "parando por N minutos em X" or "fazer uma paragem para WC no caminho, perto de X".
- When a Supplier and a separate Location / Address Note both describe the same stop, don't repeat the place name twice — pick whichever reads more naturally as the destination.
- When two rows share the same exact `Time` (e.g. a drop-off immediately followed by a luggage drop at a different location), render them as consecutive lines without repeating the time, exactly as the reference does for 12:15 on Day 1 ("Deixar os clientes primeiro perto do Jardim público" / "Depois, deixar as malas no MAR DE AR AQUEDUTO").
- Write the whole document in European Portuguese, matching the reference exactly — this is an operational document for Tinto's own on-the-ground team and a Portugal-based transport supplier, distinct from the English guest-facing materials this marketplace otherwise produces (and distinct from Sabrina's own English-language preference for what Claude says to her — that governs conversation and other deliverables, not this document's actual content, which has always been in Portuguese).

## Format and layout

Reproduce the reference document's layout exactly — this is a plain internal working document, not one of Tinto's branded guest-facing PDFs, so **do not** apply `tinto-brand.md`'s palette, typefaces, or logo here. No color, no logo, no decorative elements.

```
Tinto Travels   [N] pessoas                    [Contact 1 name]:   [phone]
Itinerário – [start day] a [end day] de [Month], [year]      [Contact 2 name]:   [phone]

Dia [n]: [DD] de [Month] ([dia da semana])
[HH:MM]  [line 1 for this time]
         [line 2, if another row shares this time — no time repeated]
[HH:MM]  [next entry]

Dia [n+1]: ...
```

- "Tinto Travels" bold; the pax count underlined next to it.
- Each "Dia N: ..." day header underlined.
- Day-of-week and month names in European Portuguese, month capitalized (Junho, not junho), day-of-week lowercase (segunda-feira, terça-feira, quarta-feira, quinta-feira, sexta-feira, sábado, domingo).
- Build the actual file with the pdf skill (read its SKILL.md before generating) — the formatting here is plain text layout (bold, underline, simple two-column header), not a styled document, so keep the pdf skill's own defaults minimal rather than reaching for a template.

## Escalate rather than guess

If a day is missing entries entirely, a time is missing where one is clearly needed, the tour's `On-Tour Contacts` is blank, or a row's `Notes` still marks it as an unconfirmed guess, say so plainly and ask Tamara rather than inventing a plausible-looking schedule — a driver working from a wrong or incomplete schedule is a real operational failure, not just an inconvenience.
