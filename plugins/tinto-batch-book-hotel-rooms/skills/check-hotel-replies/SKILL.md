---
name: check-hotel-replies
description: >
  This skill should be used when running the scheduled daily check for
  hotel replies on pending room-hold requests, or when Tamara asks "did any
  hotels reply," "check for hotel responses," "run the hotel reply check,"
  or wants to know why a particular Hotel Room Inventory row is still
  Requested. It reads Gmail threads on pending requests, interprets what
  the hotel actually said, and proposes the resulting Hotel Room Inventory
  update for Tamara to confirm — it never writes to Airtable on its own.
metadata:
  version: "0.1.0"
---

# Checking for Hotel Replies

This closes the loop `batch-book-hotel-rooms`'s Half 1 opens: instead of Tamara having to remember to check her inbox and then narrate the hotel's reply back to Claude for Half 1.5, this skill does the checking and the reading itself. It draws the same line `daily-supplier-communications` draws around sending email — draft, don't send — except here the line is around Airtable: **read and interpret, but never write without Tamara's yes.** A wrong write here doesn't just sit in a Notes field; it cascades into `Room Blocks` and a tour's real sellable capacity, so the one thing this skill must never do is decide on her behalf.

## The mechanism, in outline

1. Pull `Hotel Room Inventory` rows with `Status` = **Requested** that carry a `[DRAFTED <date> THREAD:<id>]` marker in Notes (written by `batch-book-hotel-rooms`'s Half 1 when the request went out). No marker means this row predates the thread-tracking addition — skip it silently rather than guessing at a thread; it'll need Tamara to trigger Half 1.5 the old way, by telling Claude directly.
2. Group rows by thread ID — one request can cover several weeks, so several rows can share one thread, and one hotel reply can confirm some of them and decline others.
3. For each distinct thread:
   a. **Confirm the request was actually sent, not just drafted.** Search Gmail for a sent message in that thread after the `[DRAFTED ...]` date (same mechanism `module1-scheduler-spec.md` established for the `[SENT <date>]` marker elsewhere in this base). If it's still sitting in Drafts, there's nothing to check yet — skip it for now, and don't report it as "no reply" in the end-of-run ping. That would misleadingly read as "the hotel is silent," when actually nobody sent anything yet; see "The ping," below, for how to flag this instead.
   b. Once sent, check for any new inbound message in that thread from the hotel's own address, after the sent date.
   c. No new inbound message → nothing to do this run for this thread. Don't ping about every quiet thread — only report what's actually new.
   d. A new inbound message exists → read it in full (`get_thread`, `PLAIN_TEXT`) and interpret it against the row(s) this thread belongs to.

## Reading and interpreting a reply

- Match the reply's content against exactly what was requested for each row sharing this thread: hotel, week(s), room count, category. Work out, per week: confirmed, declined, or unclear.
- If the reply changes the room count or category from what was originally asked (offered fewer rooms, a different category), note that specifically — don't assume the original ask still stands just because the hotel said yes to the week.
- If a reply is genuinely ambiguous, partial, or conditional ("we can probably do two of the three, will confirm the third by Friday") — say so plainly in the proposal rather than picking a confident-sounding interpretation. Name what's clear and what isn't.
- If a reply doesn't actually look like it's about this request (a different topic landed in the same thread, e.g. an invoice question) — don't force an interpretation onto it. Say plainly that this message doesn't read as a reply to the room request, and leave the row alone.
- This reading is where judgment belongs — same trust this skill's design already places in reading a hotel's cancellation policy or rate offer elsewhere in this base. What doesn't belong here is deciding: see below.

## Propose, then wait

For every thread with a genuine reply to report, phrase it as a plain proposal, not a completed action. For example: *"Mar de Ar Aqueduto replied on the pending request for 16–19 Jun 2028 — looks like they confirmed 12 and 19 June, declined 26 June. Want me to update Hotel Room Inventory accordingly?"* Include enough of the actual reply's substance that Tamara could sanity-check the reading herself without opening Gmail, but keep it to the point — this is a proposal to react to, not a report to study.

Wait for her answer in the same conversation before touching Airtable. Her reply is what actually triggers the write, using exactly `batch-book-hotel-rooms`'s existing Half 1.5 mechanism — this skill never re-implements that logic, it only supplies the trigger:

- **A plain "yes" (or equivalent)** → apply Half 1.5 as written: confirmed weeks → `Available` + `Cancellation Deadline` computed from the hotel's confirmed policy + `[CONFIRMED <date>]` Notes marker; declined weeks → `Not available` + `[DECLINED <date>]` marker. Same mechanism whether the interpretation came from Claude reading the email or from Tamara paraphrasing it herself.
- **A correction** ("actually it's the 19th and 26th, not the 12th and 19th") → use her correction in place of the original reading, then proceed the same way.
- **No answer yet, or she wants to check the email herself first** → leave the row untouched. Don't write anything, and don't re-propose the same thread again next run while it's still awaiting her answer — see the marker below.

## Marking that this was proposed, so repeat runs don't nag

Once a reply has been read and proposed, append `[REPLY PROPOSED <date>]` to the row's Notes — a signal to future runs of this skill, not a status change Tamara needs to see.

- A future run finds this marker followed by a `[CONFIRMED ...]` / `[DECLINED ...]` marker → the proposal was acted on. Skip it.
- A future run finds this marker with nothing after it, and no message newer than the one already proposed → still awaiting her answer. Don't re-propose the same reading every day.
- A future run finds a message in the thread newer than the last `[REPLY PROPOSED ...]` → genuinely new (she asked a follow-up question and the hotel answered that too). Read and propose the new one.

## The ping

Same hard requirement as `daily-supplier-communications`'s end-of-run ping: this always ends with a plain-language summary in Tamara's own chat, even when there's nothing to report — "no hotel replies today" is a complete, valid ping; a silent run is not.

Cover, in a short, scannable form:
- Every thread with a new reply: the proposed reading, ready for her yes / no / correction.
- Any thread still awaiting an answer from a previous run's proposal — a one-line reminder, not a full re-proposal.
- Any pending request whose email doesn't look sent yet (still in Drafts) — flag this separately; it's a different problem ("nobody sent it") from "the hotel hasn't replied yet," and conflating the two would misdirect her.
- Any thread that came back ambiguous — name what's clear and what isn't.

## What's hers to change, and what isn't

Same boundary as `batch-book-hotel-rooms`: Tamara can act on any proposal this skill surfaces, correct a misreading, or defer one — ordinary day-to-day judgment, no permission needed. She should not change the confirm-then-write boundary itself (asking for replies to be applied automatically, without her sign-off) without Sabrina's sign-off — that boundary exists because a wrong write here cascades into `Room Blocks` and a tour's real sellable capacity, not because it's an arbitrary rule.

## Escalate, don't guess, when you see:

- A reply that doesn't map cleanly onto the rows it should correspond to (a different room count than any week that was actually requested, or mentions a week nobody asked about).
- A thread with more than one plausible reply-worthy message and no clear way to tell which is the operative one.
- The same thread producing a new-looking reply *after* it's already been confirmed/declined via Half 1.5 — that's a possible correction from the hotel after the fact, not a fresh request. Flag it rather than silently reopening or overwriting a row that's already `Available`, `Not available`, or `Sold`.

In all of these cases: stop, don't write anything, and say plainly in the ping what looked wrong and why.
