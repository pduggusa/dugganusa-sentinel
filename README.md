# DugganUSA Microsoft Sentinel Connector

**Ingest 1.5M+ threat indicators into Microsoft Sentinel via TAXII 2.1. One-click deploy.**

## What's New in 1.3.0

**The liveness loop is now wired** — new Logic App playbook `dugganusa-report-hit` reports matched indicator entities back to the feed-efficacy axis. See [Reporting hits (liveness axis)](#reporting-hits-liveness-axis) below.

The DugganUSA feed now exposes **four live, no-auth validation endpoints** so your SecOps team can independently verify feed quality before operationalizing it. They are durable across our platform deploys and each response carries a `source` field (`live` | `durable` | `baseline`):

- **Novelty** — [`/api/v1/feed-uniqueness`](https://analytics.dugganusa.com/api/v1/feed-uniqueness): ~75%+ of our independently-sourced IOCs are **not** in ThreatFox.
- **Timeliness** — [`/api/v1/kev-lead`](https://analytics.dugganusa.com/api/v1/kev-lead): a live ledger of how far ahead of CISA KEV we flagged each exploited CVE — positive leads, same-day, and no-receipt all shown honestly, with receipts.
- **Accuracy** — [`/api/v1/spamhaus-validation`](https://analytics.dugganusa.com/api/v1/spamhaus-validation): Spamhaus independently corroborates our first-hand contributions.
- **Liveness** — [`/api/v1/feed-efficacy`](https://analytics.dugganusa.com/api/v1/feed-efficacy): opt-in consumer reports of when our indicators actually fire on real traffic — proof the feed is operationally live, not just large.

As a feed consumer, you can report hits to the liveness axis with the included `dugganusa-report-hit` playbook (see below) — privacy-preserving, only the matched indicator is sent, never victim data.

Feed depth also grew with **OSV malicious-package feeds (npm + PyPI)** and **daily GitHub Hunt detections**, drawn from 15 external feed sources plus our own first-hand telemetry.

> **Important:** the TAXII feed is now **API-key-enforced**. Anonymous requests return `401`; unregistered keys return `429`. The free tier is a **free registered key** — [register here](https://analytics.dugganusa.com/stix/register) before deploying.

## Deploy

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpduggusa%2Fdugganusa-sentinel%2Fmain%2FSolution%2FData%2520Connectors%2Fdugganusa-taxii-connector.json)

Or deploy via CLI:

```bash
az deployment group create \
  --resource-group YOUR_RG \
  --template-file "Solution/Data Connectors/dugganusa-taxii-connector.json" \
  --parameters workspaceName=YOUR_WORKSPACE apiKey=dugusa_YOUR_KEY
```

## What It Does

- Connects Sentinel to the DugganUSA TAXII 2.1 endpoint
- Polls every 1/6/12/24 hours (configurable)
- Populates the `ThreatIntelligenceIndicator` table
- Indicators auto-correlate with CommonSecurityLog, Syslog, AzureActivity, etc.
- Includes KQL hunting queries for IOC matching and new C2 detection

## TAXII 2.1 Endpoint

```
Server:     https://analytics.dugganusa.com/api/v1/stix-feed/taxii2
Collection: dugganusa-threats
Auth:       api-key / dugusa_YOUR_KEY (registered key required)
```

## Included Hunting Queries

- `dugganusa-ioc-match.kql` — correlate firewall/proxy logs against DugganUSA IOCs
- `dugganusa-new-c2.kql` — alert on connections to newly-indexed C2 infrastructure

## Reporting hits (liveness axis)

The playbook **`Solution/Playbooks/dugganusa-report-hit/azuredeploy.json`** closes the **Liveness** axis. Deploy it as a Microsoft Sentinel automation playbook (Logic App), then attach it to an analytics rule (or run it on-demand from an incident). On an incident it extracts the IP and URL indicator entities and POSTs them to `POST /api/v1/feed/hit` (`consumer_kind: sentinel`).

```bash
az deployment group create \
  --resource-group YOUR_RG \
  --template-file "Solution/Playbooks/dugganusa-report-hit/azuredeploy.json" \
  --parameters DugganUSAFeedKey=dugusa_YOUR_KEY
```

The feed key is a `securestring` parameter; the Sentinel connection authenticates with the playbook's managed identity.

**Privacy:** the playbook maps **only** indicator entity values (IP `Address`, URL `Url`) into the report — never Account, Host, Mailbox, or any victim-side entity.

## Free API Key

A registered key is required — the feed rejects anonymous requests. Register a free key (500 queries/day) at [analytics.dugganusa.com/stix/register](https://analytics.dugganusa.com/stix/register).

## Part of the DugganUSA Ecosystem

- [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=DugganUSALLC.dugganusa-threat-intel)
- [Splunk TA](https://github.com/pduggusa/dugganusa-splunk)
- [CLI Tool](https://github.com/pduggusa/dugganusa-cli)
- [GitHub Action](https://github.com/pduggusa/dugganusa-action)
- [Chrome Extension](https://github.com/pduggusa/dugganusa-chrome)
- [Slack Bot](https://github.com/pduggusa/dugganusa-slack)
- [STIX Feed](https://analytics.dugganusa.com/api/v1/stix-feed)
- [dugganusa.com](https://www.dugganusa.com)

## License

MIT — [DugganUSA LLC](https://www.dugganusa.com)

---

<!-- DUGGANUSA-FAMILY-FOOTER-V1 -->
## DugganUSA Defender Family

Same threat corpus, surfaced wherever you live. Open source, MIT licensed, receipts on every repo.

| Plugin | Surface |
|---|---|
| [dugganusa-scanner-core](https://github.com/pduggusa/dugganusa-scanner-core) | Core IOC scanning engine |
| [dugganusa-vscode](https://github.com/pduggusa/dugganusa-vscode) | VS Code extension |
| [dugganusa-splunk](https://github.com/pduggusa/dugganusa-splunk) | Splunk Technology Add-on |
| [dugganusa-slack](https://github.com/pduggusa/dugganusa-slack) | Slack bot |
| [dugganusa-raycast](https://github.com/pduggusa/dugganusa-raycast) | Raycast extension |
| **dugganusa-sentinel** _(this repo)_ | Microsoft Sentinel TAXII connector |
| [dugganusa-obsidian](https://github.com/pduggusa/dugganusa-obsidian) | Obsidian plugin |
| [dugganusa-nvim](https://github.com/pduggusa/dugganusa-nvim) | Neovim plugin |
| [dugganusa-elastic](https://github.com/pduggusa/dugganusa-elastic) | Elastic / OpenSearch integration |
| [dugganusa-edge-shield](https://github.com/pduggusa/dugganusa-edge-shield) | Cloudflare Worker |
| [dugganusa-cli](https://github.com/pduggusa/dugganusa-cli) | CLI scanner |
| [dugganusa-chrome](https://github.com/pduggusa/dugganusa-chrome) | Chrome extension |
| [dugganusa-action](https://github.com/pduggusa/dugganusa-action) | GitHub Action |
| [dredd-mcp](https://github.com/pduggusa/dredd-mcp) | Pre-flight MCP security (this repo) |

Backed by the live DugganUSA threat intel platform: [analytics.dugganusa.com](https://analytics.dugganusa.com).

_Jeevesus saves. Dredd judges._
