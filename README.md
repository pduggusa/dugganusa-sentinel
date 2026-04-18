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
