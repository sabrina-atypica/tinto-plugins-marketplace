# Bus itinerary formats by destination

The plain day-by-day driver schedule this skill produces is **not one universal template**. Tamara provided five real examples (2026-09-17) covering five destinations, and the language, header shape, and level of detail genuinely differ between them. Look up the tour's `Location` (from `Tours`), find the matching section below, and follow that destination's own conventions — don't default to the Alentejo format for a destination that isn't Alentejo.

None of these five source documents correspond to a tour currently in production Airtable — all five are dated 2026, and every tour in the production base as of 2026-09-17 is dated 2027. They're kept here purely as format/convention references, not as data tied to a specific `Tours` record. If you're ever asked to produce a schedule for one of these same trips for real, treat it as a fresh pull from Airtable, not a reprint of the source `.docx`.

**General rule across every destination:** these are real working drafts from Tamara, not polished output — some have inconsistent time formatting or a missing pax count. Flag a genuine content inconsistency like that to Tamara rather than silently "fixing" it your own way, and rather than treating everything in a draft as necessarily correct just because it's the reference. (One thing that looked like an inconsistency isn't: the Slovenia source's filename and body text disagree on the year, but since these documents are format templates only — never data for a specific tour — that particular mismatch doesn't matter and isn't worth raising again.) In particular, always prefer `Tours.Guests Booked` for the pax count over any number that appears in a supplier-facing note — the Philip Carter Alentejo example has a header that says "14 pessoas" and a first-line note that says "25 pessoas + MALAS." Those two numbers disagreeing is exactly the kind of thing to flag, not average out or guess between.

## Alentejo — European Portuguese

Source: `BUS 2026 06 08 Linganore alentejo.pdf`, `BUS 2026 06 22 Philip Carter alentejo.docx`.

- Header, two columns: left side "Tinto Travels" (bold) + pax count (underlined, "[N] pessoas"); right side one or two on-tour contacts, "[Name]: [phone]".
- Second line: "Itinerário – [start day] a [end day] de [Mês], [ano]".
- Day header (underlined): "Dia [n]: [DD] de [Mês] ([dia da semana])" — dia da semana lowercase, Mês capitalized.
- Entries: plain "[HH:MM]  [description]" lines, no bullets, no drive distances. Two entries at the same time are consecutive lines with the time shown only once.
- Language: European Portuguese throughout, plain operational phrasing ("saída para X", "voltar para o hotel", "pick-up em X").

## Loire Valley — French

Source: `BUS 2026 07 06 Tinto Loire.docx`.

- Header: "[start DD] [month] à [end DD] [month] [year]" alongside "[N] personnes + bagages"; then one "contact: [Name] ([phone])" line per contact.
- Day header: "Day [n]: [French weekday], [DD] [month]" — note the English word "Day" is kept even though the rest of the document is French; don't translate it to "Jour."
- Entries: "[HH:MM]  [description]", frequently followed by a drive distance/duration, e.g. "départ pour Orléans   133km (2h)". A full street address gets its own line under the stop it belongs to.
- A day with no bus activity can be shown as just the day header with a blank/separator line rather than omitted — don't invent stops to fill it.
- Language: French.

## Northern Adriatic & Slovenia — English

Source: `BUS 2026 09 21 Tinto Slovenia .docx`. (Its filename says 2026 and its body says 2027 — that mismatch is irrelevant here: this document is a pure format template, not data for a specific tour. Confirmed with Sabrina 2026-09-24; when a real Slovenia tour comes up, pull its actual dates fresh from Airtable and never reuse anything date-related from this source file.)

- Header block: tour name as a caps title line; "Dates: [Weekday Month DD] – [Weekday Month DD], [year]"; "Contact: [Name] Tel.: [phone]"; "Nr. of passengers: [N] participants + [luggage note]".
- Day header: "Day [n] – [Weekday]".
- Entries: bulleted, "· [HH:MM] – [description], [full street address]".
- A day the bus genuinely isn't needed says so explicitly ("all day in Trieste and bus will not be needed") rather than being left blank or omitted.
- Language: English.

## Southern Tuscany & Umbria — English

Source: `BUS 2026 10 05 Tuscany Miami II.docx`.

- Header: "TINTO TRAVELS [DESTINATION] – [Month DD] – [Month DD], [year]"; "Contact: [Name] [phone]". This example has **no pax count at all** — ask Tamara whether that's a real omission worth fixing or just this draft, rather than assuming a Tuscany schedule never needs one.
- Day header: "Day [n] – [Weekday], [DD Month]".
- Entries: "[HH:MM] – [description]" (en dash, no bullet); an address is inlined into the same line when given, not on its own line like Loire.
- Language: English.

## Castille & León — Spanish

Source: `BUS 2026 10 12 District Pit Spain .docx`.

- Header info (pax, contact, date range) appeared at the very end of Tamara's draft, and the contact phone number was cut off ("Tania Rodrigues t:+") — that's a document artifact to fix, not a convention to copy. Put the header at the top of the actual output, and confirm the missing digits with Tamara rather than guessing them: "[Name]: [phone]", "[N] personas", "semana: [start DD] a [end DD] de [mes], [año]".
- Day header: "Día [n]: [día de la semana] [DD] de [mes] [año]".
- Entries: "[HH:MM]  [description]  [distance] ([duration])"; multiple stops in a day are numbered inline, "1ª parada: ...", "2ª parada: ...".
- Luggage flag uses Tamara's own shorthand, "COM TODA SU EQUIPAJE" (Portuguese "COM" folded into the Spanish sentence) — keep it as she writes it rather than correcting it to pure Spanish.
- Language: Spanish.

## No reference yet for a destination

If a tour's `Location` isn't one of the five above (e.g. Porto & Douro, Coastal Tuscany, Peloponnese, Puglia, Douro, Vinho Verde, Austria), don't guess a format by analogy to the closest-sounding one — ask Tamara for a real example first, the same way this reference itself got built.
