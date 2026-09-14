# Tinto Booking Pages

Audits a tour's Airtable data against what its public booking page actually needs, then publishes it — a standalone plugin, not part of Nélia's `tinto-guest-communication`.

## Overview

This plugin covers the readiness audit and publish step for a tour's public sign-up/booking page: checking every field the live page depends on, proposing a specific, sourced fix for any gap rather than just flagging it, and publishing via the production Worker's `/api/publish-tour` endpoint once everything's confirmed. It was originally built as part of `tinto-guest-communication`, then deliberately split out into its own plugin (2026-09-14) — booking-page publishing is not currently going out to Nélia as an update to her plugin; this is a separate install for whoever handles it (Sabrina, Peter, or Nina).

## Components

| Skill | Purpose |
|---|---|
| `booking-page-publishing` | Audits a tour's Airtable data against booking-page requirements — Slug, pricing, guest-facing copy, photos, linked Client/Itinerary Days/Packages — proposing a specific, sourced fix for every gap rather than a bare pass/fail list, then publishes live via `/api/publish-tour` (Airtable + a shell `curl` call, no GitHub or repo access needed) and verifies the result. Includes an explicit warning about the current guest-payment-to-Airtable webhook issue that must be confirmed fixed before any first-time publish. |

No agents or hooks — the skill's own escalate-rather-than-guess guidance covers the judgment calls (ambiguous data, non-200 responses, an unconfirmed webhook fix).

## Setup

Requires the org's existing **Airtable** connector. Publishing itself needs only Airtable access and the standard Cowork shell (for one `curl` call) — no device bridge, repo access, or GitHub required.

## Usage

Ask Claude things like:
- "Is [tour]'s booking page ready?" — runs the readiness audit and proposes fixes for any gap.
- "Publish [tour]'s booking page" — audits and, where possible, publishes.
- "Why isn't [tour] bookable yet?" — troubleshoots using the same skill.

## Not in this plugin

Guest and winery communications, client itinerary PDFs, and winery-record entry — Nélia's `tinto-guest-communication` plugin, a separate install in the same marketplace. Supplier communications, rooming lists, and transport-company itineraries — Tamara's `tinto-logistics` plugin.
