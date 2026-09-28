# Classification examples and edge cases

Worked examples for each category in `guest-inbox-triage`'s SKILL.md, and the judgment calls between neighboring categories. When a real email doesn't clearly match one of these patterns, that's itself a signal to lean toward Category E/F (flag) rather than force it.

## Post-Tour Winery Approval replies (checked before the six categories)

- "Yes, happy for you to reach out to our travelers about next year — go ahead!" → clear yes; propose `Post-Tour Preference — Winery Approval` = Approved — Send to Travelers, `Post-Tour Preference — Approval Date` = the date this email was sent.
- "Not this year, we haven't decided on next year's dates yet." → clear no; propose Declined, same date rule.
- "Let me check with the team and get back to you." → hedged, no decision yet — Category F, not a guess. Don't propose anything.
- A winery reply that arrives when the Client has two Tours both sitting at Pending Winery Approval, and the email doesn't say which trip it's about → Category F — flag it and name both Tours, rather than assuming it's about the more recent one.
- The same Client's Tours are all at Not Yet Asked or already Approved/Declined (nothing Pending) when a winery email comes in → there's no outstanding request to match; classify the email through the normal six categories instead (most likely A, C, or E depending on content).

## Category A — No action needed

- "Thanks so much, see you in September!"
- "Got it, thank you!"
- An automatic "Out of Office" or mail-delivery bounce.
- A reply that only confirms something already known ("Yes that address is right") with no new fact and no question.

## Category B — Structured data volunteered

- "My phone number is actually 555-0143, not what I gave you before." → propose updating `Participants.Phone` for the matched traveler.
- "I'm vegetarian, forgot to mention that." → propose `Participants.Food Restriction`.
- "My return flight is UA882, landing at 6:40pm on the 14th." → propose `Participants.Departure Info`.
- "Would it be possible to get twin beds instead of sharing a double?" → propose `Participants.Bed Preference` = "Twin Beds" (once that field/form question is live).

**Judgment call, B vs. D:** if it maps to an actual field on `Participants` or `Reservations`, it's B. If it's a real request but there's no field for it (see D's examples), it's D — don't stretch a field's meaning to force a fit (e.g. don't write "wants a window seat on the bus" into `Bed Preference`).

## Category C — Answerable from real data on file

- "What time does the bus leave on the first day?" → check `Itinerary Days`/`Bookings (Confirmations)` for that Tour, draft the real answer.
- "Is breakfast included at the hotel?" → check the linked Supplier record.
- "What's the address of [hotel]?" → check the Suppliers record's address field.
- "Can you remind me what's included in the package price?" → check the Tour/Package's own guest-facing description fields.

**Judgment call, C vs. E:** if answering confidently requires information that isn't actually in Airtable (a policy question, something that depends on circumstances Claude can't see, anything where a wrong answer would matter), that's E, not a best-guess C. "Can I bring my dog" is E, not C, even though it sounds like a simple question — Tinto's actual policy on this isn't a field anywhere.

## Category D — Ad hoc request, no structured field

- "We'd love to be roomed near the Andersons if possible."
- "Any chance of a late checkout on the last day?"
- "My travel companion has a bad knee — is there a lot of walking on day 3?" (the walking-difficulty question might get a Category C answer if the itinerary describes it; the room/logistics implication, if any, is D.)

## Category E — Sensitive or out-of-lane

- "I need to cancel my trip, what's your refund policy?" → Peter/Nina's territory (finance), flag plainly.
- "We're really unhappy with how our deposit was handled." → complaint, flag, no drafted reply.
- "Can you ask the hotel if they have a room with a bathtub?" → this is a request *to* the hotel, which is Tamara's lane (supplier communication), not something to draft a guest reply to or write into Airtable as if it were internal.
- "I heard from another guest that the itinerary changed — is that true?" → could be sensitive/reputational; flag rather than draft a reassuring-sounding answer that might not be accurate.

## Category F — Ambiguous / low-confidence

- An email that mixes a simple question with something clearly emotional or complaint-adjacent — don't try to draft a partial reply to the easy part; flag the whole thing.
- A message forwarded from someone else, or clearly not written by the guest themselves, where intent isn't clear.
- Anything in a language the classification can't confidently read.
