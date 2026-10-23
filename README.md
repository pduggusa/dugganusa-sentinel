# DugganUSA Microsoft Sentinel Connector

**You do you. I do me. We share contexts.**

Your Sentinel workspace keeps its data; the connector pulls the DugganUSA Feed over TAXII 2.1 and shares nothing back.

**Ingest the DugganUSA Feed into Microsoft Sentinel via TAXII 2.1. One-click deploy.** Corpus size: [live counts](https://analytics.dugganusa.com/api/v1/search/stats).

## What's New in 1.4.0 (2026-10-19): verified in a real Sentinel workspace

We deployed this connector into our own Microsoft Sentinel workspace and checked what actually arrived. It found real defects, now fixed:

- **The template's API root returned 404 for indicators.** Sentinel requests `{API root}/collections/{id}/objects/`; with the old values that path did not exist, so a one-click deployment imported nothing. The template now uses `https://analytics.dugganusa.com/api/v1/taxii/` and collection `dugganusa-threat-intel`. If you set up a connector by hand with the old values (`/api/v1/stix-feed/taxii2`, `dugganusa-threats`), it keeps working: the old path now serves the same objects.
- **Hunting queries read the wrong table.** Sentinel writes TAXII indicators to `ThreatIntelIndicators`; the older `ThreatIntelligenceIndicator` table stays empty for them, so `dugganusa-ioc-match` found nothing and `dugganusa-new-c2` failed to run. Both are rewritten and were tested against our live indicators.
- **The Tor query used `externaldata()` against a URL that now requires a key**, so it silently returned nothing. It now reads a Sentinel watchlist you fill from the keyed API.
- **The one-click deployment itself failed.** Azure routed the old connector resource to a 2020 API version that rejects the TAXII connector kind. The template now uses the current resource type, fills in the workspace id itself, defaults to once-a-day polling and a 7-day import (free-tier safe), and installs the hunting queries in the same step.
- **On our side of the feed:** invalid STIX ids, confidence values of 100 and `ip:port` values in IPv4 indicators were fixed in what we serve. Details: [analytics.dugganusa.com/updates](https://analytics.dugganusa.com/updates).

## What's New in 1.3.0

**The liveness loop is now wired** — new Logic App playbook `dugganusa-report-hit` reports matched indicator entities back to the feed-efficacy axis. See [Reporting hits (liveness axis)](#reporting-hits-liveness-axis) below.

The DugganUSA feed now exposes **four live, no-auth validation endpoints** so your SecOps team can independently verify feed quality before operationalizing it. They are durable across our platform deploys and each response carries a `source` field (`live` | `durable` | `baseline`):

- **Novelty** — [`/api/v1/feed-uniqueness`](https://analytics.dugganusa.com/api/v1/feed-uniqueness): how much of our independently sourced data is not already in ThreatFox, so you can see what this feed adds beside the feeds you already run.
- **Timeliness** — [`/api/v1/kev-lead`](https://analytics.dugganusa.com/api/v1/kev-lead): a live ledger of when each exploited CVE entered our corpus next to its CISA KEV listing date, with receipts, including the ones we had no record of.
- **Accuracy** — [`/api/v1/spamhaus-validation`](https://analytics.dugganusa.com/api/v1/spamhaus-validation): Spamhaus independently corroborates our first-hand contributions.
- **Liveness** — [`/api/v1/feed-efficacy`](https://analytics.dugganusa.com/api/v1/feed-efficacy): opt-in consumer reports of when our indicators actually fire on real traffic — proof the feed is operationally live, not just large.

As a feed consumer, you can report hits to the liveness axis with the included `dugganusa-report-hit` playbook (see below) — privacy-preserving, only the matched indicator is sent, never victim data.

Feed depth also grew with **OSV malicious-package feeds (npm + PyPI)** and **daily GitHub Hunt detections**, drawn from 15 external feed sources plus our own first-hand telemetry.

> **Important:** the TAXII feed is now **API-key-enforced**. Anonymous requests return `401`; unregistered keys return `429`. The free tier is a **free registered key** — [register here](https://analytics.dugganusa.com/stix/register) before deploying.

## Deploy (one step, two inputs)

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpduggusa%2Fdugganusa-sentinel%2Fmain%2FSolution%2FData%2520Connectors%2Fdugganusa-taxii-connector.json)

You need a workspace with Microsoft Sentinel enabled and a DugganUSA API key ([free](https://analytics.dugganusa.com/stix/register)). Pick the workspace's resource group, type the workspace name, paste the key. That's it. The deployment creates:

- the **Threat intelligence - TAXII** connector for our feed, polling **once a day** and importing the **last 7 days** by default, which fits the free tier (choose hourly with a paid key);
- three hunting queries under **Hunting**: *What arrived (feed health)*, *IOC match (firewall and proxy logs)* and *New C2 in the last 6 hours*.

After the first poll, open **Hunting** and run **DugganUSA - What arrived (feed health)**. It counts indicators by type with their newest arrival and highest confidence (never above 95).

Prefer the CLI? The key is a `securestring`, so the CLI prompts for it instead of taking it on the command line:

```bash
az deployment group create \
  --resource-group YOUR_RG \
  --template-file "Solution/Data Connectors/dugganusa-taxii-connector.json" \
  --parameters workspaceName=YOUR_WORKSPACE
```

## What It Does

- Connects Sentinel to the DugganUSA TAXII 2.1 endpoint
- Polls every 1/6/12/24 hours (configurable)
- Populates the `ThreatIntelIndicators` table (Sentinel's current threat-intelligence table)
- Indicators auto-correlate with CommonSecurityLog, Syslog, AzureActivity, etc.
- Includes KQL hunting queries for IOC matching and new C2 detection

## TAXII 2.1 Endpoint

```
API root:   https://analytics.dugganusa.com/api/v1/taxii/
Collection: dugganusa-threat-intel
Auth:       any username, your API key as the password (registered key required)
```

Manual setup in the portal (Content hub > Threat Intelligence > data connector **Threat intelligence - TAXII**) takes the same values. The friendly name accepts letters, digits, `-` and `_` only.

## Included Hunting Queries

- `dugganusa-ioc-match.kql`: firewall and proxy logs (CommonSecurityLog) against DugganUSA IPs and domains, with a confidence floor you set (default 80)
- `dugganusa-new-c2.kql`: connections to command-and-control IPs that arrived in the last 6 hours
- `dugganusa-tor-relay-match.kql`: connections to or from Tor exit relays, from a watchlist

All three were run in our own workspace on 2026-10-19; the first two matched synthetic log rows built from our live indicators.

## Reporting hits (liveness axis)

The playbook **`Solution/Playbooks/dugganusa-report-hit/azuredeploy.json`** closes the **Liveness** axis. Deploy it as a Microsoft Sentinel automation playbook (Logic App), then attach it to an analytics rule (or run it on-demand from an incident). On an incident it extracts the IP and URL indicator entities and POSTs them to `POST /api/v1/feed/hit` (`consumer_kind: sentinel`).

```bash
az deployment group create \
  --resource-group YOUR_RG \
  --template-file "Solution/Playbooks/dugganusa-report-hit/azuredeploy.json"
```

The CLI prompts for `DugganUSAFeedKey` (a `securestring`).

The feed key is a `securestring` parameter; the Sentinel connection authenticates with the playbook's managed identity.

**Privacy:** the playbook maps **only** indicator entity values (IP `Address`, URL `Url`) into the report — never Account, Host, Mailbox, or any victim-side entity.

## Free API Key

A registered key is required — the feed rejects anonymous requests. Register a free key (10 queries/day; [pricing](https://analytics.dugganusa.com/pricing)) at [analytics.dugganusa.com/stix/register](https://analytics.dugganusa.com/stix/register).

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

Same DugganUSA Feed, surfaced wherever you live. Open source, MIT licensed, receipts on every repo.

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
| [dredd-mcp](https://github.com/pduggusa/dredd-mcp) | Dredd: pre-flight MCP security |

Backed by the [DugganUSA Feed](https://analytics.dugganusa.com/stix). Numbers are live: [/api/v1/search/stats](https://analytics.dugganusa.com/api/v1/search/stats). Pricing: [analytics.dugganusa.com/stix/pricing](https://analytics.dugganusa.com/pricing). Contact: help@dugganusa.com.

_Threat intelligence you can check._
