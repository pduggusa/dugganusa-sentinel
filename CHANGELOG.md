# Changelog

All notable changes to the DugganUSA Microsoft Sentinel Connector are documented here.

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
