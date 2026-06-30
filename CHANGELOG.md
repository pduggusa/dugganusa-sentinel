# Changelog

All notable changes to the DugganUSA Microsoft Sentinel Connector are documented here.

## [1.3.0] - 2026-06-30

### Added
- **Feed-efficacy hit reporting (liveness loop)** — new playbook `Solution/Playbooks/dugganusa-report-hit/azuredeploy.json` (Logic App, ARM). On a Microsoft Sentinel incident it extracts IP and URL indicator entities and POSTs them to `POST /api/v1/feed/hit` (`consumer_kind: 'sentinel'`), closing the Liveness validation axis (`/api/v1/feed-efficacy`). Feed key is a `securestring` parameter; the Sentinel connection uses managed identity.
- **Privacy contract:** the playbook maps ONLY indicator entity values (IP `Address` / URL `Url`) into the report — never Account, Host, Mailbox, or any victim-side entity. (The platform also drops any victim-side field server-side.)

## [1.2.1] - 2026-06-30

### Added
- Documented the fourth live validation axis — Liveness (`/api/v1/feed-efficacy`) — alongside novelty, timeliness, and accuracy. Consumers can opt in to report hits via `POST /api/v1/feed/hit` (privacy-preserving — only the matched indicator is sent, never victim data).

### Changed
- Refreshed IOC corpus copy to 1.5M+ IOCs (~1.57M live).
- Reworded the Timeliness validation bullet to point at the live kev-lead ledger instead of a fixed "~31 days ahead" average (the live ledger is the source of truth).

## [1.2.0] - 2026-06-27

### Added
- Documented the three live, no-auth, durable feed-validation endpoints — novelty (`/api/v1/feed-uniqueness`), timeliness (`/api/v1/kev-lead`), and accuracy (`/api/v1/spamhaus-validation`) — for independent SecOps verification. Each response carries a `source` field (`live` | `durable` | `baseline`).
- Noted new feed depth: OSV malicious-package feeds (npm + PyPI) and daily GitHub Hunt detections.

### Changed
- **TAXII feed is now API-key-enforced.** The `apiKey` ARM parameter is now required (no longer defaulted to blank); anonymous requests return `401` and unregistered keys return `429`. Removed all "blank for free tier" / "works without one" copy.
- Corrected feed metadata to 1.10M+ IOCs (was 1,080,000+).
- Bumped ARM template `contentVersion` to 1.2.0.0.

## [1.1.0]

- TAXII 2.1 data connector, KQL hunting queries (IOC match, new C2, Tor relay match), one-click Azure deploy.
