---
name: daily-supplier-communications
description: >
  This skill should be used when Tamara asks to "check today's supplier
  emails," "run the daily supplier communications check," "see what's due
  today," "draft the supplier emails," "what needs to go out today," or asks
  how the daily scheduler works, why a particular supplier was or wasn't
  emailed, or what to do with a drafted email. It covers the full supplier
  communications cycle: checking Airtable for what's due, drafting into
  Gmail (English-first, thread-aware), the human review/send step, and the
  end-of-run ping.
metadata:
  version: "0.4.0"
---

# Daily Supplier Communications

Guide Tamara through Tinto Travels' automated supplier-booking email cycle. This skill covers one channel only — **suppliers** (hotels, wineries-as-venue, restaurants, transport, tour guides/cultural sites): never guests or wineries-as-sales-clients, which is Nélia's lane in a separate plugin, out of scope here even if asked about.

## The mechanism, in outline

A scheduled process checks the production Airtable base's `Supplier Booking Lead Times` table once a day, plus the calendar-anchored Season Reconfirmation sweep and the capacity-anchored Hotel Final Confirmation trigger. Each row (or sweep) that's newly due today gets drafted into Gmail. `logistics-reference.md` (project doc) is the live reference for supplier categories, touchpoint timing, and per-supplier fields — read it live rather than trusting a fixed list here, since these get revised without notice.

When asked to run this check (or when picking up a scheduled run):

0. **List tours that are sold but not linked to suppliers.** Any Tour with `Status` = Confirmed and no `Bookings (Confirmations)` rows at all has just been sold and handed over, and nothing in this table exists for it yet, so no supplier will ever be contacted for it until it's linked. Also list any tour whose `Bookings (Confirmations)` rows carry `[LINKED AT HANDOVER BY TAMARA` but whose emailable, non-hotel suppliers have no `Supplier Booking Lead Times` rows (a linking run that stopped halfway). These go at the top of the ping (see "End of every run"). Don't link them here; that's the `link-tour-suppliers` skill in `tinto-confirm-supplier-pairings`, run with Tamara.
1. **Read live, don't assume.** Pull `Supplier Booking Lead Times` and cross-reference the linked `Tour` and `Supplier` records directly in the production base. If unsure which field is the current trigger for a touchpoint, check `logistics-reference.md` or ask Tamara rather than guessing.
2. **Identify what's due today** — First Contact (rolling, per category lead time; hotels excluded, see `batch-book-hotel-rooms`), Final Reconfirmation (28 days out), Room Count Reconciliation and Hotel Final Confirmation (hotel-only, see `logistics-reference.md`) — for every row whose `Booking Request Status` is "Not Due Yet" (or blank — treat blank the same way) and whose due condition is now met. **Season Reconfirmation is a separate check, not a row check — see its own section below.** It runs every day alongside this one, but it isn't triggered by any row's due date, so don't look for it in the row sweep above.
3. **Check the pairing is actually confirmed before drafting to it** — see "Gate: don't draft to an unconfirmed pairing," below. This is a hard check, not a nice-to-have: a meaningful share of this table was built by matching a tour's region against a supplier's region, not from a confirmed itinerary, and drafting to the wrong supplier is a worse outcome than a short delay.
4. **Draft each one that passes the gate** — see "How to draft," below. This is where all three of Tamara's feedback items apply.
5. **Mark the row** — move it to "Due — Draft Needed" then "Drafted" once the Gmail draft exists, same as the existing scheduler convention. Do this per-row, not in bulk, so a failure partway through doesn't leave a row silently un-drafted.
6. **End every run with the ping** — see "End of every run," below. This is not optional and not a summary you write only when something interesting happened — it is the entire point of this skill from Tamara's side.

## Season Reconfirmation — a calendar sweep, not a row check (run this every day too)

Season Reconfirmation is genuinely different from every other touchpoint in this skill: it's anchored to the calendar (November 3rd / April 1st), not to any individual `Supplier Booking Lead Times` row's lead-time days. There is no `(Season Reconfirmation)` row waiting to come due — checking for one, the way step 2 above checks for First Contact or Final Reconfirmation rows, will never find anything, because none exist. This section is the actual mechanism; it's ported directly from `module1-scheduler-spec.md` §1b (the original dry-run design, built 2026-08-04) and adapted for production — that design was never carried over when this plugin was first built, which is a real gap this section closes.

Run this check every day, right alongside the row-based check above, in this order:

1. **Determine the two candidate trigger dates that have already passed.** The most recent November 3rd on or before today (this year's, if today's on or after Nov 3, otherwise last year's), and the most recent April 1st on or before today (same logic). This gives a built-in catch-up: a scheduler outage or a gap before the scheduled task existed doesn't silently skip a whole season.
2. **Spring check.** The Spring season being reconfirmed = March–July of (that November 3rd's year + 1). Dedup key = `SEASON_RECONFIRMATION_SPRING_<that year+1>`. Look this Key up in production's **Automation Config** table (`Key`/`Value`/`Notes` fields). If a record with this Key already exists, that Spring sweep already ran — skip it entirely. If it does not exist, the sweep is due: find every Confirmed Tour with Start Date in that March–July window, and for each, every distinct Hotel/Winery/Restaurant/Transport supplier already linked to it (any existing `Supplier Booking Lead Times` row for that Tour, any Label, deduped by Supplier — Tour Guides are out of scope, same as Final Reconfirmation). Run the same pairing gate as everywhere else in this skill (a "Draft estimate — needs confirming" pairing with no `[PAIRING CONFIRMED...]`/`[PAIRING CORRECTED...]` marker, whether by Tamara, at handover or by the ops sheet, still doesn't get drafted here either) and the same Language Preference / placeholder-email checks from "How to draft." Draft a lighter-touch "still on for these dates, more detail closer to the time" email for each qualifying (Tour, Supplier) pair — not a booking request, and not the Final Reconfirmation headcount/dietary ask.
3. **Fall check.** Same logic, Fall season = September–November of that April 1st's year (same year, not +1). Dedup key = `SEASON_RECONFIRMATION_FALL_<that year>`.
4. **If neither Spring nor Fall is due today** — the common case almost every day of the year — say so in one line in the ping and move on. Don't pad the summary.
5. **After every draft in a sweep succeeds:** append `[SEASON RECONFIRMED SPRING-<year> <today>]` (or `FALL-<year>`) to the Notes of every `Supplier Booking Lead Times` row belonging to that (Tour, Supplier) pair — same append-only convention, and same "mark every row in the pairing" pattern `confirm-supplier-pairings` already uses, not just one. Then write one new Automation Config record: `Key` = `SEASON_RECONFIRMATION_SPRING_<year>` (or `FALL_<year>`), `Value` = today's date, `Notes` = which tours/suppliers this sweep covered.
6. **Safety rule — only mark a sweep complete once it's actually complete.** Write the Automation Config record only after every applicable draft for that sweep has succeeded. If something fails partway through (a placeholder email, a tool error, a gated pairing), do not write the completion record — leave it unwritten so tomorrow's run retries the whole sweep, and name in the ping which suppliers still need drafts. A sweep marked "done" when it wasn't would silently skip suppliers for a full year.

**A real gap this surfaced, flagged rather than quietly worked around:** the design above (and `logistics-reference.md`) both say Season Reconfirmation and Final Reconfirmation share the same scope — Hotel/Winery/Restaurant/Transport. But production's `Supplier Booking Lead Times` table has no `(Final Reconfirmation)` rows for Hotel at all — the 2026-09-09 bulk wire only created them for Winery/Restaurant/Transport. Hotels still have other rows for this same sweep to anchor to (Room Count Reconciliation, Rooming List, Final Confirmation), so the Season Reconfirmation sweep above still works correctly for them — but it's worth Sabrina/Peter/Nina confirming whether Hotel Final Reconfirmation rows were meant to exist and are simply missing from production, or whether hotels were always meant to skip that specific touchpoint (they already get three others). Don't create those rows unilaterally; just don't assume the current production shape is the intended one.

## Gate: don't draft to an unconfirmed pairing

Every row in `Supplier Booking Lead Times` has a `Confidence` field. Before drafting anything for a row that's due:

- If `Confidence` is **"Confirmed with supplier"**, **"Confirmed by Peter/Nina (internal process)"**, or **"Peter's rule of thumb"** — proceed normally, draft as usual.
- If `Confidence` is **"Draft estimate — needs confirming"**, check the row's `Notes` field for any of these markers:
  - `[PAIRING CONFIRMED BY TAMARA ...]` or `[PAIRING CORRECTED BY TAMARA ...]`, written by the `confirm-supplier-pairings` skill after Tamara reviews a pairing.
  - `[LINKED AT HANDOVER BY TAMARA ...]`, written by the `link-tour-suppliers` skill when Tamara links a newly sold tour to its suppliers (copied from the most recent tour at the same destination, confirmed by her).
  - `[SUPPLIER CHANGED BY TAMARA ...]`, written when she corrects a supplier at drafting time (see "When Tamara says a supplier is wrong for a tour," below).
  - `[PAIRING CONFIRMED BY OPS SHEET ...]`, written when this table is synced to `Bookings (Confirmations)`. It means the supplier appears on that tour in `Bookings (Confirmations)`, which is verified against Nélia's ops reservation workbook for that destination. It confirms **who** the supplier is, not the lead time. It was first applied in bulk on 2026-09-25.
  - Marker present → the pairing has already been reviewed and confirmed (or corrected — the `Supplier` link will already point at the right record). Proceed normally. The row's original Notes text may still say "matched by region only, not yet confirmed" above the marker. Notes are append-only, so the later marker is what counts.
  - No marker → **do not draft.** This pairing was built by matching the tour's region against the supplier's region, not from a confirmed itinerary, and hasn't been checked by a human yet. Leave `Booking Request Status` as it is (don't move it to "Due — Draft Needed" — nothing was drafted), and instead add it to a distinct section of the end-of-run ping: "awaiting pairing confirmation." Name the tour and the supplier, and ask Tamara whether this supplier is really on this tour. If she confirms, append `[PAIRING CONFIRMED BY TAMARA <today>]` to every row of that (Tour, Supplier) pair; if she says it's a different supplier, follow "When Tamara says a supplier is wrong for a tour."

This gate applies every run. New tours no longer start with guessed pairings: they get no rows at all until Tamara links them with `link-tour-suppliers` (step 0 lists them), and those rows carry the handover marker. Rows without any marker are leftovers from the original region-matching and from the 9 tours that have no ops sheet.

## When Tamara says a supplier is wrong for a tour

New tours are linked by copying the most recent tour at the same destination, so the moment a draft is about to go to a supplier is when Tamara is most likely to notice "not this restaurant for this tour, we use X." When she says that (while reviewing a run, or about a draft already in Gmail), fix the data, not only the email. The bus itinerary and every later touchpoint read these tables, so an email-only fix leaves them wrong.

1. **Find the replacement in `Suppliers`.** If it has no record, stop: don't invent one. Say so, leave everything as it is, and flag that the supplier needs adding first.
2. **Bookings (Confirmations):** for this Tour, every row whose Supplier is the old supplier. Confirm with Tamara which of those stops change (usually all of them; ask if the old supplier appears on several days). Update `Supplier` to the new one and append `[SUPPLIER CHANGED BY TAMARA <today>: was <old>, now <new>]` to Notes.
3. **Supplier Booking Lead Times:** every row for this (Tour, old supplier). Update `Supplier`, replace the old name with the new one in `Label`, and append the same marker to Notes. If the new supplier's Type is a different category from the old one (for example a restaurant replaced by a winery lunch), don't guess the lead times: say so and ask, since lead-time rules are Peter/Nina's.
4. **Check what already went to the old supplier.** If any of the old supplier's rows for this tour are already `Drafted` or `Sent`, tell Tamara plainly: the old supplier may be expecting this group. Offer to draft a short cancellation or change note to them (draft only, same rules as every other supplier email). Never assume one isn't needed.
5. **Replace the draft.** If a Gmail draft to the old supplier exists for this touchpoint, delete that draft (`delete_draft`), then draft to the new supplier following "How to draft" (thread check included), and mark the row `Drafted` as usual.
6. **Confirm back** exactly what changed: which Bookings rows, which lead-time rows, which draft was replaced, and whether a note to the old supplier is pending.

Only the tour Tamara names changes. If she says the change applies to all future tours at this destination, still change only this tour's rows; the next tour linked by `link-tour-suppliers` copies from the most recent tour, so the change carries forward on its own.

## How to draft

### 1. Always draft in English first — this overrides the per-supplier Language Preference field at the drafting stage

Every supplier email is written in English first, full stop, regardless of what that supplier's `Language Preference` field says. This is Tamara's own explicit feedback, and it's a real, deliberate change from how the Module 1 scheduler spec originally worked (that spec checked `Language Preference` *before* drafting and wrote directly in the target language).

- `Language Preference` still matters — it's just moved to a later step. It tells you what the *eventual sent* email's language should be, once Tamara has reviewed and approved the English content.
- If a supplier's `Language Preference` is English (or blank, defaulting to whatever the standing per-category default is), there's nothing further to do — the English draft is the final draft.
- If a supplier's `Language Preference` is anything other than English, say so plainly when you report the draft: name the supplier, name the target language, and flag that translation is a follow-up step she can ask for once she's happy with the English wording — **do not translate pre-emptively as part of the same drafting pass.** She reviews the substance in the language she actually works in before anything gets converted.
- Never silently skip this note. A German-preference or Portuguese-preference supplier whose draft doesn't mention translation at all reads as "this is already right for sending," which it isn't yet.

### 2. Check for an existing Gmail thread before creating any draft — never create a disconnected new email if one already exists

Before calling `create_draft`, always check whether there's already an open conversation with this supplier that the new message should continue:

1. Search Gmail with `search_threads` using the supplier's email address (`to:<supplier email> OR from:<supplier email>`) plus a relevant keyword if useful (tour name, season, or the general topic — e.g. "availability" or "booking"). Keep the query loose enough to actually find a real prior thread; an over-specific query will miss one that exists under a slightly different subject line.
2. If a matching thread turns up, call `get_thread` on it (use `PLAIN_TEXT` format to keep this efficient) to confirm it's genuinely the same ongoing conversation with this supplier — not a coincidental match — and to get the ID of its most recent real message.
3. If it's a real match, create the draft with `create_draft`'s `replyToMessageId` set to that most recent message's ID. This threads the new draft into the existing conversation instead of starting a new, disconnected one — exactly what Tamara asked for.
4. If no real matching thread exists, create a plain new draft as normal (no `replyToMessageId`).

This check happens every time, for every supplier email this skill drafts — first contact, reconfirmations, follow-ups, everything. A follow-up or reconfirmation email is especially likely to have a prior thread; treat a follow-up that isn't threaded as a mistake to go back and fix, not a minor slip.

### 3. Content and voice

Match the established Tamara-signed, professional style already built into the supplier-email templates (`Tinto-Alentejo-Supplier-Emails-Review.docx` and the equivalent per-region material, referenced from `logistics-reference.md`). Pull the actual tour dates, headcount, and supplier details from the linked Tour/Supplier/Bookings (Confirmations) records so every draft is genuinely specific, not templated placeholder text. Sign every draft **Tamara**. Never call any Gmail send capability for a supplier email, under any circumstances — drafting only. Sending is Tamara's deliberate human step.

After drafting, append a dated marker to the row's Notes field (`[DRAFTED <date>]`), same convention the scheduler has always used, so future runs and the no-reply escalation logic (see `logistics-reference.md`) know when this happened.

### 4. WhatsApp-preferred suppliers

If the supplier's `Preferred Communication Channel` is WhatsApp (or the draft otherwise needs to move outside Gmail), still draft normally in Gmail first — that's still the record of what was said — but flag clearly, every time, that this one needs manually copying over rather than sent as an email. Don't let this rule interact with the thread-checking step above in a way that skips it — even a WhatsApp-preferred supplier's Gmail draft should still be threaded if a prior Gmail thread exists.

## End of every run — the ping

This is the second piece of Tamara's feedback, and it's a hard requirement on every run, not just the ones where something happened: **the final message of the run is a plain-language overview she can read at a glance, in her own Claude chat.** Cover, in a short, scannable form:

- Which supplier emails were due today (supplier name, tour, what kind of touchpoint — first contact, reconfirmation, etc.).
- Confirmation that the drafts now exist in Gmail — and for any of them, whether they landed inside an existing thread or as a new email, so she knows what to expect when she opens Gmail.
- **Tours sold but not yet linked to suppliers** (step 0), at the very top, every day until they're linked: tour name, destination, start date, and how long until its first touchpoint would normally fall due (transport is usually first, around 210 days out). Suggest running `link-tour-suppliers`. Also list any half-finished linking runs.
- **Any rows that were due but skipped by the pairing gate above**: name each tour/supplier pair and ask whether that supplier is really on the tour (see the gate). Don't bury this among the drafted ones; it's a different kind of item (blocked on her review, not waiting on her to send).
- Any supplier whose `Language Preference` isn't English, flagged as needing translation once she approves the content (see above) — don't bury this in the per-draft detail only, repeat it in the summary too.
- Any WhatsApp-preferred suppliers needing a manual copy-over.
- **Whether a Season Reconfirmation sweep ran today** — name the season and year if one did (and how many suppliers it covered), or say plainly "no season sweep due today" if not. If a sweep started but didn't fully complete (see the Season Reconfirmation section's safety rule), say exactly which suppliers are still pending so it's clear tomorrow's run will retry, not skip, them.
- Anything skipped (a placeholder contact email, an unrecognized row, a blocked no-reply escalation) — same escalation conventions as the rest of Module 1.
- If genuinely nothing was due today, still send this — a short, honest "nothing due today" line. A silent run is not a successful ping; the whole point is that Tamara doesn't have to go check Airtable herself to know the answer.

This overview only actually reaches her as a real "ping" if the scheduled task that invokes this skill is bound to her own account with push notifications on — see the plugin README's Setup section. If you're ever running this skill inside an ordinary chat with Tamara rather than a scheduled firing, the same overview still applies — just address it to her directly in that conversation.

## What's hers to change, and what isn't

Tamara can edit the **content and wording** of supplier email templates herself, including the underlying template, not just one draft's wording — that's established, existing scope (`module1-permission-split-draft.md`), not something this plugin needs to gate. She can also decide not to send a particular drafted email, or send it later — nothing automatic re-fires or nags once a row is marked "Drafted."

She should **not** change: the trigger timing or lead-time windows for any touchpoint, the escalation day-counts, or anything in the Airtable schema (field types, table structure, the scheduler logic itself). Those are system-wide rules that stay with Peter and Nina. If a change like that seems needed, say so plainly and suggest she raise it with them rather than making the change.

## Escalation — when to stop and flag instead of fixing it

- A supplier's email on file looks like a placeholder (contains "placeholder", ".test", or is otherwise obviously fake) — don't draft to a fake address. Leave the row at "Due — Draft Needed," flag it plainly in the end-of-run ping as blocked on a real contact.
- A drafted email would thread into what looks like the wrong conversation (a thread that matched on the supplier's email address but reads as being about something unrelated) — when in doubt, draft fresh rather than force a mismatched thread, and say so in the ping.
- A row's Label or category doesn't match anything in `logistics-reference.md`'s established touchpoint patterns — don't guess a lead time or rule; skip the row, describe it in the ping, and let Tamara or Sabrina decide.
- Anything that would mean editing the trigger timing, escalation windows, or Airtable schema — that's Peter/Nina's call, not something to patch through this skill.

In all of these cases: stop, don't send or "fix" the draft, and say plainly in the end-of-run ping what looked wrong and why.
