---
name: guest-inbox-triage
description: >
  This skill should be used whenever daily-guest-communications runs — the
  two are meant to always run together as one daily check. Triggers on
  Nélia's existing phrases ("check today's guest emails," "run the daily
  guest communications check," "see what's due today," "what needs to go
  out today") as well as "what came in," "any replies," "anything from
  guests/wineries today." It covers reading new inbound guest and winery
  email, matching each message to a known Reservation or winery contact,
  classifying what it's about, and proposing the right response — an
  Airtable update to confirm, a drafted Gmail reply to review, or a plain
  escalation flag — never an automatic send or an automatic write.
metadata:
  version: "0.1.0"
---

# Guest Inbox Triage

The inbound half of Nélia's daily communications cycle. `daily-guest-communications` covers what Tinto sends out on a schedule; this skill covers what comes back — and anything a guest or winery writes in that was never prompted by a scheduled touchpoint at all (a special request, a question, a complaint). **Always run this alongside `daily-guest-communications` as part of the same daily check** — report both halves together (see "Combined report" below), not as two separate replies.

## Why this exists

Before this skill, a guest's ad hoc email (a special request, a question, a change of plan) had no reliable path into Airtable or a reply — it depended entirely on Nélia noticing it and remembering to act, with nothing tracking whether she had. This skill removes that dependency for the common cases by making inbox review part of the same daily habit that already exists, and by being explicit about what it can safely decide on its own (nothing) versus what needs a human's yes (everything that touches Airtable or goes out the door).

## Scope: which emails to look at

**Only emails whose sender matches a known Reservation or winery contact.** Never treat this as a general inbox scan.

1. **Guest side:** sender address matches a `Reservations` record's `Buyer Email`. Resolve to that Reservation, its linked `Tour`, and its linked `Participants`.
2. **Winery side:** sender address matches a `Client / Affiliation` record's `Contact Email` (added 2026-09-28 — this field is new and likely sparsely populated at first; if a real winery email doesn't match anything because the field is still blank for that client, say so plainly in the report as a data gap worth backfilling, rather than silently skipping it as "unmatched"). Resolve to that Client/Affiliation record and its linked `Tours`.
3. **Anything else** — sender doesn't match either — is out of scope. Don't classify it, don't mention it, don't act on it. This skill only ever touches Nélia's actual lane: guests and wineries, never suppliers (that's Tamara's, per `guest-communication-reference.md`'s existing ownership boundary), and never her general inbox.

**Tracking what's already been handled:** apply a Gmail label — `Claude/Triaged` — to every message once it's been classified this way, whichever category it lands in (including "no action needed"). Each run searches matched-sender mail that doesn't have this label yet, bounded to a reasonable lookback window (last 14 days) as a safety net in case a run is missed. Don't rely on read/unread state — Nélia may open an email herself before this runs, and that shouldn't cause it to be silently skipped.

## Classification — six categories, escalate when unsure

Read each matched, unlabeled email and sort it into exactly one of these. **If a message could plausibly fit more than one, or doesn't clearly fit any, it always goes to Category E or F (flag) — never force a guess into B, C, or D.** See `references/classification-examples.md` for worked examples of each category and the judgment calls between them.

**A — No action needed.** Acknowledgments ("thanks!," "got it," "sounds good"), auto-replies, out-of-office bounces, anything with no request and no new information in it. Label it triaged. Don't write to Airtable, don't draft a reply, don't itemize it in the report — just count it (see "Combined report" below).

**B — Structured data volunteered.** The sender states something that maps to a real, existing field — a phone number, return flight/departure details, a food restriction, passport details, a bed preference (once `Participants.Bed Preference` is live on the Guest Info Form — see `guest-request-capture-skill-spec-2026-09-28.md`) — anything the Guest Info Form itself asks for, just arriving by email instead. **Never write this to Airtable automatically.** Propose the exact change — record, field, old value if any, new value — as one line in the report, for Nélia to confirm before anything is written.

**C — A question answerable from real data already on file.** "What time do we leave," "is breakfast included," "what's the hotel's address," "how do we get from the airport" — anything answerable from the Tour, Destination, Package, or Suppliers records already in Airtable. Draft a reply into Gmail (never send it — see "What this skill never does" below), sourced only from real field values. **If you can't confidently source the answer from real data, this is not a Category C — treat it as Category E instead.** A wrong or fabricated answer is worse than no drafted answer at all.

**D — An ad hoc request with no structured field.** Room-pairing asks ("we'd like to be near the Smiths"), informal logistics requests ("can we get a late checkout"), anything real but with nowhere structured to live. Propose it as an addition to that Reservation's `Notes` field, written the same careful way described in `guest-request-capture-skill-spec-2026-09-28.md` (preserve any existing `Room N.` tag at the start of Notes, append after it — this is what `rooming-lists` later reads as a footnote). Same confirm-first treatment as Category B — propose, don't write.

**E — Sensitive or out-of-lane.** Cancellations, refund or billing requests, complaints, disputes, anything emotionally loaded, or anything that's actually Tamara's (supplier-related) or Peter/Nina's (financial, per `finance-reference.md`) territory rather than Nélia's. **Flag only. No drafted reply, no proposed Airtable write, no attempt to route or forward it automatically.** Say plainly what came in, who it's from, and — when it's clearly someone else's lane — say so, but leave the actual handling to a human. A wrong-toned auto-draft on a complaint is worse than making someone write it from scratch.

**F — Ambiguous or low-confidence.** Treat exactly like Category E: flag it, explain what's unclear about it, and stop there.

## Matching and resolving — escalate, don't guess

Same discipline `rooming-lists` and the other Airtable-writing skills in this marketplace already apply:

- If an email's sender matches more than one Reservation (a shared family email used across two different bookings, for instance) or the match is otherwise ambiguous, say so in the report and ask rather than picking one.
- If a matched Reservation has more than one linked Participant and the email doesn't make clear which traveler a Category B fact belongs to, propose the write with the ambiguity called out explicitly ("unclear which of the two travelers on this Reservation this refers to — confirm before I write it") rather than guessing.
- Never write to a Reservation or Participant record you're not confident is the right one. A note attached to the wrong guest is worse than a short delay.

## Combined report

Report this alongside `daily-guest-communications`' own output as one daily check, with the inbound half added as new sections:

- **Proposed Airtable updates** (Categories B and D) — one line each: record, field, proposed value, and a yes/no.
- **Drafted replies ready to review** (Category C) — same review-before-send step Nélia already does for every scheduled touchpoint draft; this is just another source feeding the same Gmail drafts folder.
- **Flagged, no action taken** (Categories E and F) — what came in, from whom, why it's flagged, and who it actually belongs to when that's clear.
- **Nothing needed** (Category A) — a single count, not itemized.
- If any winery email couldn't be matched because `Client / Affiliation.Contact Email` is still blank for that client, say so as its own line (a data gap to backfill), separate from genuinely unmatched/out-of-scope mail.

## What this skill never does

- **Never sends an email.** Every drafted reply goes into Gmail as a draft only, exactly like every other guest/winery email in this system — Nélia reviews and sends everything herself.
- **Never commits an Airtable write without it appearing in the report for confirmation first.** Nothing in Categories B or D lands in Airtable on its own.
- **Never acts on an unmatched sender.**
- **Never drafts a reply for anything sorted into Category E or F.**
- **Never touches supplier communication**, even if a guest or winery email happens to mention a hotel or supplier by name — stays strictly guest/winery, per Nélia's existing lane boundary.
- **Never fabricates an answer** to source a Category C reply — if the real data isn't there, it's a Category E, not a guess.

## What's hers to change, and what isn't

Same split as `daily-guest-communications`: Nélia can adjust wording of drafted replies and how flagged items are described to her — she should not need permission to reword something. She should not change which categories exist, the escalation defaults, the dedup/labeling mechanism, or anything about when something is allowed to write to Airtable without confirmation — those are structural decisions Sabrina made when this was designed, not day-to-day wording.
