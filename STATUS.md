# Plugin status

Where each plugin actually stands, at a glance. Full detail and history lives in each plugin's own
README.md, and in `claude/plugin-marketplace-build.md` and `claude/rooming-lists-and-git-migration.md`
in the Tinto Travels project. Update this table whenever a plugin's status changes, regardless of which
session does it, so it never depends on any one session's memory.

**Status legend:**
- **Done and verified**: rebuilt against live production data and confirmed correct, either through a
  direct simulation or a real live use.
- **Design rebuilt, not yet confirmed end to end**: works against live schema and real records in every
  check run so far, but has not yet been watched succeed on a full real world send (to a hotel, a
  supplier, a guest).
- **Original placeholder**: not yet redesigned since this repo was first assembled.
- **In progress on a branch**: someone is actively rebuilding it right now, not yet merged to master.

| Plugin | Version (master) | Status |
|---|---|---|
| `tinto-guest-communication` | 0.7.0 | Done and verified. Restricted to Nelia's daily workflow. |
| `tinto-logistics` | 0.8.0 | Done and verified for the daily cycle; 0.8.0 changes merged 2026-09-25 and not yet watched on a real run. Restricted to Tamara's daily workflow. 0.8.0: the drafting gate accepts `[PAIRING CONFIRMED BY OPS SHEET]` (540 rows marked in the 2026-09-25 sync), `[LINKED AT HANDOVER BY TAMARA]` and `[SUPPLIER CHANGED BY TAMARA]`; every run lists Confirmed tours with no Bookings (Confirmations) rows as "sold, not yet linked"; new path for correcting a supplier at drafting time that updates both tables and replaces the draft. |
| `tinto-confirm-supplier-pairings` | 0.3.0 | Rebuilt 2026-09-25 and merged: no longer an audit of region-matched guesses, now the sales to ops handover skill `link-tour-suppliers` (copies the most recent tour at the same destination into Bookings (Confirmations) and Supplier Booking Lead Times after one confirmation from Tamara). Dry run only so far; not yet used on a real tour (first candidate: Linganore Alentejo, Oct 4 2027). Manual JSON/frontmatter check only, `claude plugin validate` unavailable from Claude's shell. |
| `tinto-batch-book-hotel-rooms` | 0.5.0 | Done and verified. Hotel Room Inventory's Status expanded to 4 stages (Requested/Available/Not available/Sold) to match the real batch-request-before-any-tour-exists process; Half 2 now bridges a sold conversion into Room Blocks (explicit complimentary-rooms question, net-capacity math, multi-hotel-sequence append). New `check-hotel-replies` skill reads a hotel's Gmail reply and proposes the Status update, but never writes without Tamara's confirmation. Simulated end to end against live production data (a real Alentejo hotel, full Requested -> Sold -> Room Blocks bridge cycle), test records cleaned up afterward. Manual schema/frontmatter validation passed (`claude plugin validate` CLI unreachable from this session's shell). Merged to master 2026-09-24. Still pending: the actual `check-hotel-replies` scheduled task, which needs to be created from Tamara's own account once the plugin's installed there. |
| `tinto-rooming-lists` | 0.6.0 | Design rebuilt and manually tested against two real tours (Alentejo, Puglia). Not yet confirmed end to end with Tamara or a live hotel send. |
| `tinto-bus-company-itineraries` | 0.6.0 | Done and verified. Rebuilt against a real reference document and new Airtable fields, extended to five destination formats, and tested end to end with a full simulated data population and PDF output against a real 2027 tour. Includes a 2026-09-24 onboarding check-in for Tamara (which destinations have templates, which real running tours still need one). `claude plugin validate` passed. Merged to master 2026-09-24. |
| `tinto-booking-pages` | 0.1.0 | Design rebuilt around the real `/api/publish-tour` endpoint, not yet watched succeed end to end on a real write. |
| `tinto-add-winery-record` | 0.3.0 | Done and verified live (real client record, Vinia Wine and Kitchen). Updated 2026-09-24 to match the booking-page team's contract for this table (logo/color validation criteria, explicit wine-club discount interview step). `claude plugin validate` passed. Ready for handover, merged to master. |
| `tinto-client-itinerary-pdf` | 0.12.0 | Done and verified twice against real data (field by field, and a live simulated request) for its original design: destination reference-itinerary reuse, new-client/new-tour hand-off, booking-page-summary framing. Updated 2026-09-25: asks Peter/Nelia directly for Price per Person and a single-supplement figure, anchored against same-destination precedent, whenever real Packages don't exist yet, since that's the normal case for a PDF that's itself the sales pitch before a tour sells; previously this silently omitted the price. That specific fallback, including the `[SINGLE SUPPLEMENT CONFIRMED ...]` Notes marker it writes on `Tours`, has not yet been exercised live in this exact written form; the live-simulated behavior it formalizes has been. Merged to master 2026-09-25. |
| `tinto-add-new-tour` | 0.6.0 | Done and verified live 2026-09-25: the full Room Block-to-Packages happy path was exercised end to end for the first time (hotel hold requested, confirmed, sold, bridged into `Room Blocks`, then `Packages` created from confirmed same-destination precedent pricing), against a real winery (The Urban Winery) and real hotel (Mar de Ar Aqueduto). All simulation records and one test Gmail draft were deleted afterward, confirmed clean. Step 3 now states explicitly that nearly every field it asks for feeds the booking page directly, and preserves a template's literal `{client}` merge token instead of overwriting it (an earlier simulation round mistook that token for an unfilled placeholder and "fixed" it, wrongly). Not yet exercised: the branch where Step 6 reuses a price Peter already confirmed for this same tour's own earlier pitch-stage PDF rather than a fresh precedent lookup, since no pitch-stage PDF happened to run ahead of the sale in this cycle's test; the fresh-lookup fallback itself has now been verified live twice. Merged to master 2026-09-25. |

**Open branches not yet in this table's "master" versions above:**
- (none currently tracked here as of 2026-09-25 -- `add-new-tour-initial-build` and `client-itinerary-pdf-delegate-winery-record` both merged to master this session.)

Check `git branch -a` in the repo for the current list, since this section can go stale faster than the
table above.
