# 🚨 Alertmanager Setup

> Enterprise Alertmanager cluster configuration for routing, grouping, deduplicating, and delivering security alerts to PagerDuty, Slack, and Jira.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Routing Tree](#routing-tree)
- [Receivers](#receivers)
- [Inhibition Rules](#inhibition-rules)
- [Silencing](#silencing)
- [High Availability](#high-availability)

---

## Overview

Alertmanager receives alerts from all Prometheus instances, deduplicates them across the HA pair, groups related alerts, and routes them to the correct team and channel based on severity, security domain, and business hours. This folder contains the full Alertmanager configuration and documentation for the enterprise cybersecurity observability deployment.

---

## Folder Structure

```
alertmanager-setup/
├── alertmanager.yml            # Main Alertmanager configuration
├── templates/
│   ├── slack-message.tmpl      # Slack notification template
│   └── pagerduty-message.tmpl  # PagerDuty notification template
├── silences/
│   └── maintenance-silence.yml # Pre-defined silence templates
└── README.md
```

---

## Configuration Files

### `alertmanager.yml`

The main configuration controls the full alert lifecycle — from receiving a firing alert to delivering it to the correct destination.

Key sections:

| Section | Purpose |
|---|---|
| `global` | SMTP relay, Slack API URL, PagerDuty URL defaults |
| `route` | Decision tree for routing alerts to receivers |
| `receivers` | Notification destinations (PagerDuty, Slack, Jira) |
| `inhibit_rules` | Suppress lower-severity alerts when a critical fires |
| `templates` | Message formatting for each channel |

---

## Routing Tree

Alerts are routed based on the `severity` and `team` labels attached at the Prometheus rule level.

```
root receiver: security-ops-slack
│
├── severity = critical
│   ├── team = security-operations   ──▶  PagerDuty (24x7 on-call)
│   │                                      + Slack #sec-alerts-critical
│   └── team = platform-security     ──▶  PagerDuty (platform on-call)
│                                          + Slack #platform-alerts-critical
│
├── severity = high
│   └── all teams                    ──▶  Slack #sec-alerts-high
│                                          + Jira auto-ticket
│
├── severity = medium
│   └── all teams                    ──▶  Slack #sec-alerts-medium
│
└── severity = low
    └── all teams                    ──▶  Jira auto-ticket only
```

### Group Settings

| Label | Group Wait | Group Interval | Repeat Interval |
|---|---|---|---|
| critical | 30s | 2m | 1h |
| high | 1m | 5m | 4h |
| medium | 2m | 10m | 12h |
| low | 5m | 30m | 24h |

---

## Receivers

### PagerDuty

Routes `critical` alerts to the security on-call rotation. Includes full alert details: affected tool, severity, description, and direct runbook link. Auto-resolves in PagerDuty when the alert clears in Prometheus.

### Slack

Three dedicated channels mapped to severity levels. Message template includes:

- Alert name and description
- Affected security tool or component
- Current value and threshold
- Runbook link
- Silence and acknowledge quick-action links

### Jira

`high` and `low` alerts automatically create Jira issues in the `SEC` project with:

- Issue type: Security Incident (high) or Task (low)
- Priority mapped from alert severity
- Component tag based on the `team` label
- Auto-close on alert resolution

---

## Inhibition Rules

Inhibition prevents alert storms by suppressing child alerts when a parent condition is already firing.

| Source Alert | Suppresses |
|---|---|
| `NodeDown` | All exporter alerts from that node |
| `PrometheusTargetDown` | All alerts from the affected scrape target |
| `VaultSealed` | Vault token expiry and lease alerts |
| `EDRAgentOffline` (critical) | EDR detection count drop alerts |

---

## Silencing

Pre-defined silence templates are stored in `silences/` for planned maintenance windows. Apply a silence via the Alertmanager API or Grafana's silence management UI.

Common silence scenarios:

- **Scheduled patching** — suppress all alerts for a target host during maintenance
- **Planned scanner runs** — suppress vuln spike alerts during a scheduled scan
- **Certificate renewal** — suppress TLS expiry alerts during rotation

---

## High Availability

Alertmanager runs as a **3-node cluster** with gossip-based state synchronization. This ensures:

- Alerts are deduplicated even when received by multiple Prometheus replicas
- No duplicate PagerDuty pages or Slack messages during failover
- Any single node failure does not interrupt notification delivery

Nodes peer with each other via the `--cluster.peer` flag and share notification state continuously.

---

> **On-Call Escalation:** If a critical alert is not acknowledged in PagerDuty within 15 minutes, it escalates automatically to the secondary on-call and then to the Security Engineering manager.
