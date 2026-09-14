---
name: guest-itinerary-pdf
description: >
  This skill should be used when Nélia asks to "create the itinerary PDF
  for [tour]," "put together [guest]'s trip itinerary," "make the
  day-by-day schedule for [tour]," or needs the specific, dated day-by-day
  itinerary document for a booked tour — the attachment referenced by the
  `daily-guest-communications` skill's trip-logistics touchpoints.
metadata:
  version: "0.1.0"
---

# Guest Itinerary PDF

Produce the guest-facing, day-by-day itinerary for one **specific booked tour** — dated, with the real suppliers/hotels actually confirmed for that departure — as a branded PDF attachment.

## Not the same thing as the Pre/Post-Tour Planning Guide

Tinto also has a separate `tinto-prepost-tour-guide` skill that builds a **destination-level** reference guide (packing tips, arrival logistics, car rental, "surprises & tips," etc.) — evergreen content that's the same for every guest going to that destination, not tied to one tour's specific dates. This skill is different: it's the **tour-specific day-by-day schedule** (Day 1: pickup at X, winery visit at Y, lunch at Z...) for one actual departure. A guest typically receives both documents at different points in the journey — don't conflate the two, and don't try to build one from the other. If a request is ambiguous about which is wanted, ask.

## Where the data comes from

**Primary source of truth: the production Airtable base's `Itinerary Days` table**, filtered to the requested tour, in `Day #` / `Time Slot` order. Each row carries the guest-facing narrative (`Guest-Facing Title`, `Guest-Facing Description`) already written in marketing voice — use this content directly rather than rewriting it, unless Nélia asks for a change. Pull the tour's own header details (name, dates, destination, linked winery/client) from the `Tours` record.

Check `Itinerary Days` directly for the specific tour rather than assuming it matches the destination's usual pattern — a given tour can diverge from the standard schedule once real bookings are in. If a day looks like it's missing, or content looks like a placeholder rather than real copy, flag it to Nélia rather than inventing filler.

## Document structure

- Cover/header: tour name, dates, destination, guest name(s) if producing a personalized copy for one reservation (check whether Nélia wants one shared PDF for the whole departure or a personalized one per guest — ask if unclear).
- One section per day, in order: day number and date, guest-facing title, guest-facing description.
- Closing section: Tinto contact information / emergency contact, matching the convention used in Tinto's other guest-facing documents.

## Branding

Build the PDF using the pdf skill (read its SKILL.md before generating), styled with Tinto Travels' guest-facing document brand — see `references/tinto-brand.md` for the palette, typefaces, and layout conventions already established (same brand system used for the Pre/Post-Tour Planning Guide). This is guest-facing, so the fuller treatment applies: logo, section styling, and the line-icon/decorative touches used elsewhere in Tinto's guest documents, not the more restrained operational-document treatment used for internal/supplier paperwork.

## Language and currency

Same rule as every other guest-facing surface: American English, no exceptions by destination. Any pricing mentioned (rare in an itinerary, but check) is USD. If the day's content includes a `Regional Sign-Off`-style destination-language flourish, that's a deliberate touch, not something to translate further or remove.

## After building

Deliver the finished PDF the normal way. If this itinerary is being produced to attach to a `daily-guest-communications` touchpoint draft, say so plainly and remind whoever's sending the email that it still needs to be attached by hand — the Gmail draft itself never carries it.

## Escalate rather than guess

If `Itinerary Days` for the requested tour is empty, clearly incomplete, or looks like unconfirmed placeholder content, say so and ask Nélia rather than producing a PDF with gaps or invented content — a guest reading a wrong or missing day is a real trip-planning failure, not just a formatting issue.
