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
| `tinto-logistics` | 0.7.0 | Done and verified. Restricted to Tamara's daily workflow. |
| `tinto-confirm-supplier-pairings` | 0.1.0 | Done and verified, carried over unchanged from Tamara's rebuilt logistics work. |
| `tinto-batch-book-hotel-rooms` | 0.2.0 | Design rebuilt against live production schema, not yet confirmed end to end. In progress on a branch (`batch-book-hotel-rooms-notes-and-scope`), adding Notes narrative writes and scoping out Supplier Payments, heading to 0.3.0. |
| `tinto-rooming-lists` | 0.6.0 | Design rebuilt and manually tested against two real tours (Alentejo, Puglia). Not yet confirmed end to end with Tamara or a live hotel send. |
| `tinto-bus-company-itineraries` | 0.1.0 | Original placeholder on master. In progress on a branch (`bus-company-itineraries-airtable-rebuild`), rebuilding it against a real reference document and making it destination aware. Not yet merged. |
| `tinto-booking-pages` | 0.1.0 | Design rebuilt around the real `/api/publish-tour` endpoint, not yet watched succeed end to end on a real write. |
| `tinto-add-winery-record` | 0.3.0 | Done and verified live (real client record, Vinia Wine and Kitchen). Updated 2026-09-24 to match the booking-page team's contract for this table: logo/color validation criteria, and an explicit wine-club discount interview step. |
| `tinto-client-itinerary-pdf` | 0.1.0 | Done and verified twice against real data (field by field, and a live simulated request). |

**Open branches not yet in this table's "master" versions above:**
- `batch-book-hotel-rooms-notes-and-scope`
- `bus-company-itineraries-airtable-rebuild`

Check `git branch -a` in the repo for the current list, since this section can go stale faster than the
table above.
