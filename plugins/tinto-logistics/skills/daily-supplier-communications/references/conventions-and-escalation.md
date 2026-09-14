# Conventions and Escalation Reference

Detail supporting the `daily-supplier-communications` skill. Load this when actually drafting or reviewing a supplier email, or when deciding whether something needs to be escalated.

## Drafting language — the English-first rule, in detail

This is the single biggest behavior change from the original Module 1 scheduler spec, so it's worth stating precisely:

- **Draft in English. Always. First.** Regardless of the supplier's `Language Preference` field, regardless of category, regardless of region.
- `Language Preference` is not ignored — it's just no longer read at drafting time. It tells you what language the *final sent* version should eventually be in, once Tamara has reviewed the English draft and is happy with its content.
- Never translate as part of the same pass that produces the first draft. Translation is a distinct, separate step, requested afterward — usually by Tamara asking something like "translate that one to Portuguese now" once she's confirmed the content is right.
- Always name the target language explicitly when a supplier's `Language Preference` isn't English — in the per-draft note and again in the end-of-run summary. Silence here reads as "this is ready to send as-is," which would be wrong.
- The known non-English cases per `logistics-reference.md`: Pretto and Fabio Gervasoni (Tuscany) are set to Italian. Greek suppliers are, per standing policy, generally set to English already (not Greek) — so they mostly won't trigger a translation flag at all, which is expected, not a bug. French suppliers are "case-by-case" per the JF decision — none are on file yet, but if one appears, treat the field the same way as any other non-English preference: draft in English first, flag for translation after review.

## Thread-matching — how to judge a "real" match

`search_threads` can return false positives (the same supplier email used for something unrelated, or a shared inbox address). Before treating a found thread as the one to reply into:

- Check the thread's subject/snippet reads as plausibly the same topic (booking, availability, a specific tour/season) — not a random unrelated exchange that happens to share an email address.
- Prefer a thread that's reasonably recent over an old, long-dead one, if more than one plausible thread exists.
- If genuinely unsure, it's safer to draft fresh (no `replyToMessageId`) and note the ambiguity in the end-of-run ping than to force a reply into the wrong conversation. A wrongly-threaded reply is a worse outcome than a slightly redundant new email — flag it rather than guess.

## Attachments

Same standing gap as the rest of this system: the Gmail connector cannot attach files. Where a supplier email should carry a day-by-day itinerary or a rooming list, draft the covering email and note in the row's Notes field (and in the end-of-run ping) that the actual document still needs to be produced/attached manually.

## Escalate, don't fix, when you see:

- A supplier's email on file that looks fake or placeholder.
- A thread match that looks wrong or ambiguous (see above).
- A `Supplier Booking Lead Times` row with a Label or category that doesn't match any established touchpoint pattern in `logistics-reference.md`.
- A tour or supplier record missing information a draft needs (no Start Date, no linked Supplier), producing a draft with blanks or clearly wrong content.
- Any situation where fixing it would mean editing the trigger timing, escalation windows, or the Airtable schema — that's Peter/Nina's call, not something to patch through this skill.

In all of these cases: stop, don't send or force a draft through, and say plainly in the end-of-run ping what looked wrong and why — then flag it to Sabrina, Peter, or Nina if it looks like it needs their decision. Guessing at a workaround risks a supplier getting a genuinely wrong or duplicated email, which is worse than a short delay while someone checks it.
