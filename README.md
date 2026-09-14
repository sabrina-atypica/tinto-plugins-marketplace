# Tinto Travels Plugin Marketplace

Self-service plugin catalog for the Tinto Travels team's Cowork accounts. Anyone on the team can add this marketplace once and then browse, install, or remove whatever plugins fit their own role — no dependency on Sabrina after that, and no need to keep asking her when a role changes.

## Plugins in this marketplace

| Plugin | For | Covers |
|---|---|---|
| `tinto-guest-communication` | Nélia (guest/winery-client comms) | Daily guest & winery emails, winery record entry, guest itinerary PDFs, booking-page readiness/publishing |
| `tinto-logistics` | Tamara (supplier operations) | Daily supplier reconfirmations, rooming lists, transport-company itineraries |

Peter and Nina can install both for oversight — nothing in either plugin grants a capability beyond what their existing Airtable/Gmail seat permissions already allow; the plugins are instructions, not an access-control layer. See each plugin's own README for what it covers and doesn't.

## How to add this marketplace (one-time, per person)

In Cowork, open **Customize → Plugins → Add marketplace**, and enter this repository's URL (or `owner/repo` shorthand if it's hosted on GitHub). Once added, both plugins above appear in **Browse plugins** — install, enable/disable individual skills, or uninstall at any time, independently of anyone else's account.

## Updating

When a plugin's skills change, push the update to this repository. Anyone who's added the marketplace can click **Update** on it (Customize → Plugins) to pull the latest version; Cowork also checks for updates automatically.

## Adding a new plugin later

Add a new folder under `plugins/`, following the same structure (`.claude-plugin/plugin.json` + `skills/*/SKILL.md`), then add an entry to `.claude-plugin/marketplace.json` pointing at it (`"source": "./plugins/<new-plugin-name>"`). No changes needed to how people already using this marketplace access it — the new plugin just appears next time they browse or update.
