---
name: daily-guest-communications
description: >
  This skill should be used when Nélia asks to "check today's guest emails,"
  "run the daily guest communications check," "see what's due today," "draft
  the guest emails," "draft the winery emails," "what needs to go out today,"
  or asks how the daily email scheduler works, why a particular guest was or
  wasn't emailed, or what to do with a drafted email. It covers the full
  guest-and-winery communications cycle: checking Airtable for what's due,
  drafting into Gmail, and the human review/send step.
metadata:
  version: "0.1.0"
---

# Daily Guest & Winery Communications

Guide Nélia through Tinto Travels' automated guest-and-winery email cycle. This skill covers two channels only — **guests and wineries** — never suppliers (hotels, restaurants, transport, cultural sites): that is Tamara's lane, in a separate plugin, and out of scope here even if asked about.

## The mechanism, in outline

A scheduled process checks the production Airtable base's `Reservations` table (and its linked `Tours` records) once a day. For each reservation, it looks at a set of trigger fields — each tied to an anchor date (Deposit Date, Tour Start Date, or Tour End Date) plus or minus a fixed number of days — to work out which of the seven guest-journey touchpoints, if any, are due today. Each touchpoint also has its own dedup field, checked before drafting, so the same email is never drafted twice for the same reservation.

When asked to run this check (or when picking up a scheduled run), do the following:

1. **Read live, don't assume.** Look up the current trigger and dedup fields directly on `Reservations`/`Tours` in the production base rather than relying on a fixed list of touchpoints or timings — these get revised without notice, and the source of truth is always the live schema, not this document. If unsure which fields are the current trigger/dedup pair for a touchpoint, check the field descriptions or ask Nélia rather than guessing.
2. **Identify what's due today.** A touchpoint is due if today matches its trigger condition for a given reservation, and its dedup field shows it hasn't already been drafted.
3. **Pull the template content from Airtable**, not from memory — templates are stored and maintained there so they can be edited without a plugin update. Personalize using the reservation's and tour's own fields (guest name, tour name, winery, dates, links, etc.).
4. **Draft into Gmail — never send.** Every touchpoint in this cycle is Gmail-drafted only. **Do not call any Gmail send capability for a guest or winery email, under any circumstances, even if the tool used to draft it also exposes a send function.** Sending is a deliberate human step — Nélia reads every draft before it goes out. The single exception is the booking confirmation email, which is not part of this cycle at all: it fires instantly and automatically through a separate system (Resend) the moment a booking completes, with no draft and no review step. If Nélia asks about a guest not receiving a confirmation, that's a different investigation from anything in this skill — it doesn't route through Gmail drafts.
5. **Mark the dedup field** once a draft is created, so the same email isn't drafted again tomorrow.
6. **Report back plainly**: which reservations got a new draft, for which touchpoint, and anything that looked ambiguous (see Escalation below).

## Reviewing and sending — Nélia's part

After drafting, Nélia reviews each draft in her own Gmail and sends it herself. Before telling her a batch is ready to review, or if asked to help her review one:

- **Check for the attachment gotcha first.** The Gmail connector cannot attach files. Several touchpoints require an attachment — the Pre-Trip Guide PDF (at Welcome and again at 6-Weeks-Out), the itinerary PDF, and the merged post-trip email draft that accompanies the Approval Request — and these are delivered separately, not inside the draft. Flag explicitly, every time, which attachment(s) a given draft needs and remind Nélia to attach it by hand before sending — this is the single easiest thing to forget, since a draft with no attachment looks complete at a glance.
- **Every guest and winery email signs "Nélia, on behalf of {Winery Name}"** — the winery name comes from the tour's linked winery record. The one exception is the winery-facing Approval Request, which is team-signed rather than personally signed by Nélia, even though drafting/sending it is still hers to do.
- Run the draft against the conventions in `references/conventions-and-escalation.md` before considering it ready — currency, language, and the destination-language sign-off line are the most common things that look like errors but aren't.

## What's hers to change, and what isn't

Nélia can edit the **content and wording** of guest/winery email templates herself — that's the same ownership Tamara has over supplier templates, since these are the actual substance of her job, not a system-wide rule. She should not need to ask permission to reword a template.

She should **not** change: the trigger timing or offset for any touchpoint, the dedup logic, or anything in the Airtable schema (field types, table structure). Those are system-wide rules that stay with Peter and Nina. If a change like that seems needed, say so plainly and suggest she raise it with them rather than making the change.

## Escalation — when to stop and flag instead of fixing it

See `references/conventions-and-escalation.md` for the full list. In short: if a trigger looks like it fired for the wrong guest, a draft looks like a duplicate of one already sent, or a reservation record looks incomplete or inconsistent in a way that would produce a wrong-sounding email, stop and flag it to Sabrina, Peter, or Nina rather than guessing at a fix or overriding the record. This is a signal something upstream needs a human decision, not something to patch in the moment.
