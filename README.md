# DugganUSA Microsoft Sentinel Connector

**Ingest 1M+ threat indicators into Microsoft Sentinel via TAXII 2.1. One-click deploy.**

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
Auth:       api-key / dugusa_YOUR_KEY (or blank for free tier)
```

## Included Hunting Queries

- `dugganusa-ioc-match.kql` — correlate firewall/proxy logs against DugganUSA IOCs
- `dugganusa-new-c2.kql` — alert on connections to newly-indexed C2 infrastructure

## Free API Key

[analytics.dugganusa.com/stix/register](https://analytics.dugganusa.com/stix/register) — works without one at reduced limits.

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
