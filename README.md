# 🔒 Prometheus Enterprise Observability

> **Enterprise-grade monitoring, alerting, and observability stack for cybersecurity infrastructure.**  
> Powered by [Prometheus](https://prometheus.io/) · Built for scale · Secured by design.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Cybersecurity Exporters & Integrations](#cybersecurity-exporters--integrations)
- [Alerting Rules](#alerting-rules)
- [Dashboards](#dashboards)
- [Security Best Practices](#security-best-practices)
- [High Availability](#high-availability)
- [Retention & Storage](#retention--storage)
- [Access Control & RBAC](#access-control--rbac)
- [Runbooks](#runbooks)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

This repository provides the **enterprise Prometheus observability stack** used to monitor all cybersecurity tooling across the organization. It enables:

- **Real-time visibility** into security tool health, performance, and signal quality
- **Unified alerting** across SIEM, EDR, WAF, IDS/IPS, vulnerability scanners, and identity systems
- **Audit-ready metrics** for compliance (SOC 2, ISO 27001, PCI-DSS, NIST CSF)
- **SLA tracking** for security operations and incident response pipelines

### Monitored Security Tool Categories

| Category | Tools |
|---|---|
| SIEM | Splunk, Elastic SIEM, Microsoft Sentinel |
| EDR / XDR | CrowdStrike Falcon, SentinelOne, Microsoft Defender |
| Vulnerability Management | Tenable.io, Qualys, Rapid7 InsightVM |
| WAF / DDoS Protection | Cloudflare, AWS WAF, Akamai |
| Identity & Access | Okta, CyberArk PAM, Azure AD |
| Network Security | Palo Alto NGFW, Cisco FTD, Zeek IDS |
| Secret & Key Management | HashiCorp Vault, AWS Secrets Manager |
| Container Security | Falco, Aqua Security, Prisma Cloud |

---

## Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          SECURITY DATA SOURCES                               │
│                                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │    SIEM     │  │   EDR/XDR   │  │  Vuln Mgmt  │  │     WAF     │        │
│  │  Exporters  │  │  Exporters  │  │  Exporters  │  │  Exporters  │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
│         │                │                │                │               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Identity   │  │   Network   │  │   Secrets   │  │  Container  │        │
│  │  Exporters  │  │  Exporters  │  │  Exporters  │  │  Exporters  │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
│         └────────────────┴────────────────┴────────────────┘               │
│                                    │  scrape over mTLS                      │
└────────────────────────────────────┼─────────────────────────────────────────┘
                                     │
┌────────────────────────────────────▼─────────────────────────────────────────┐
│                        PROMETHEUS ENTERPRISE CORE                            │
│                                                                              │
│   ┌──────────────────────┐              ┌────────────────────────────────┐  │
│   │    Prometheus A      │              │      Alertmanager Cluster      │  │
│   │    (Primary)         │──── alerts ─▶│  ┌────────────┐ ┌───────────┐ │  │
│   └──────────────────────┘              │  │   Node 0   │ │  Node 1   │ │  │
│   ┌──────────────────────┐              │  └────────────┘ └───────────┘ │  │
│   │    Prometheus B      │──── alerts ─▶│       gossip state sync        │  │
│   │    (Replica)         │              └───────────────┬────────────────┘  │
│   └──────────────────────┘                             │                   │
│              │                                         ▼                   │
│              ▼                          ┌────────────────────────────────┐  │
│   ┌──────────────────────┐              │      Notification Routing      │  │
│   │    Thanos Sidecar    │              │  PagerDuty · Slack · OpsGenie  │  │
│   │    (Block Upload)    │              └────────────────────────────────┘  │
│   └──────────┬───────────┘                                                  │
└──────────────┼───────────────────────────────────────────────────────────────┘
               │
┌──────────────▼───────────────────────────────────────────────────────────────┐
│                         STORAGE & VISUALIZATION                              │
│                                                                              │
│   ┌──────────────────────┐              ┌────────────────────────────────┐  │
│   │     Thanos Query     │──────────── ▶│       Grafana Dashboards        │  │
│   │     (Global View)    │              │  Security · Compliance · SLA   │  │
│   └──────────────────────┘              └────────────────────────────────┘  │
│              │                                                               │
│              ▼                                                               │
│   ┌──────────────────────────────────────────────┐                          │
│   │               Object Storage                 │                          │
│   │   Hot (15d) ──▶ Warm (90d) ──▶ Cold (1yr+)   │                          │
│   │       S3 / GCS      ·      Glacier / Nearline │                          │
│   └──────────────────────────────────────────────┘                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| Prometheus | v2.50+ | LTS recommended |
| Alertmanager | v0.27+ | Clustered mode |
| Grafana | v10.3+ | For dashboards |
| Thanos / Cortex | Latest stable | For HA & long-term storage |
| Kubernetes | K8s 1.28+ | Production deployment target |
| TLS Certificates | — | mTLS between all components |

---

## Cybersecurity Exporters & Integrations

All security tools expose metrics via dedicated exporters that Prometheus scrapes on a defined interval over mTLS.

| Exporter | Port | Scrape Interval | Metrics Provided |
|---|---|---|---|
| crowdstrike-exporter | 9118 | 60s | Agent health, detection counts, policy status |
| sentinelone-exporter | 9119 | 60s | Threat detections, agent connectivity, rollback events |
| tenable-exporter | 9120 | 120s | Vulnerability counts by severity, scan completion rates |
| falco-exporter | 9376 | 30s | Runtime security events, rule match rates |
| vault-exporter | built-in | 30s | Secret leases, token TTL, seal status |
| okta-exporter | 9145 | 120s | MFA failures, suspended users, policy violations |
| waf-exporter | 9121 | 30s | Blocked requests, rule triggers, bot traffic ratios |
| qualys-exporter | 9122 | 120s | Asset coverage, critical CVE counts, scan freshness |

---

## Alerting Rules

Alerts are grouped by security domain and defined with severity labels, team ownership, and runbook links. Every critical and high alert includes a suppression duration to reduce transient noise, and an annotation pointing to the operational runbook.

Key alert domains covered:

- **EDR Health** — agent coverage thresholds, detection pipeline latency
- **Vault Security** — seal status, token expiry, replication lag
- **Vulnerability Management** — critical CVE spike rates, scan staleness
- **Identity & Access** — MFA failure surges, privileged account anomalies
- **WAF / DDoS** — block rate anomalies, rule trigger spikes
- **SIEM Ingestion** — event pipeline lag, indexing failure rates
- **Container Runtime** — Falco rule matches, syscall anomaly rates

### Alert Severity Matrix

| Severity | Response SLA | Notification Channel |
|---|---|---|
| critical | 15 minutes | PagerDuty (on-call) + Slack #sec-alerts-critical |
| high | 1 hour | Slack #sec-alerts-high + Jira ticket auto-created |
| medium | 4 hours | Slack #sec-alerts-medium |
| low | Next business day | Jira ticket only |

---

## Dashboards

Pre-built Grafana dashboards are provisioned automatically and all are version-controlled in this repository.

| Dashboard | Description |
|---|---|
| cybersec-overview | Executive summary — coverage, detections, SLA health |
| edr-health | EDR agent connectivity and detection pipeline throughput |
| vuln-management | Vulnerability trends by severity and asset group |
| identity-security | MFA failures, privileged access events, account anomalies |
| vault-operations | Secret leases, token health, replication lag |
| runtime-security | Falco events, container anomalies, syscall rule matches |
| compliance-posture | Control coverage mapping for SOC 2 and PCI-DSS |
| incident-response | MTTD, MTTR, alert volume trends, on-call load |

---

## Security Best Practices

### mTLS Everywhere

All Prometheus scrape connections and Alertmanager cluster communication are secured with mutual TLS. Certificate lifecycle is managed by cert-manager with automatic rotation. The `insecure_skip_verify` flag is never set to true in any environment.

### Secrets Management

All credentials — API keys, bearer tokens, and scrape secrets — are injected at runtime via Kubernetes Secrets or the Vault Agent Sidecar. No credentials are hardcoded in configuration files or committed to the repository. Scrape tokens rotate every 24 hours via Vault dynamic secrets.

### Network Policies

Kubernetes NetworkPolicies enforce that only Prometheus pods may initiate connections to exporter pods on their metrics ports. No exporter is reachable from outside the monitoring namespace.

### Audit Logging

All Prometheus API access events and configuration change operations are forwarded to the central SIEM via the audit sidecar, providing a complete trail for compliance reviews.

---

## High Availability

Two Prometheus replicas run with hard pod anti-affinity to ensure they land on separate nodes. Thanos Sidecar ships blocks to object storage and Thanos Query provides a unified, deduplicated view across both replicas. Alertmanager runs as a three-node cluster with gossip-based state synchronization to prevent duplicate notifications.

```
  Prometheus A ──▶┐
                  ├──▶ Thanos Sidecar ──▶ Thanos Query ──▶ Grafana
  Prometheus B ──▶┘
                               │
                               ▼
                    Object Storage (S3 / GCS)
                    Long-term retention up to 1 year+
```

---

## Retention & Storage

| Tier | Duration | Storage Backend |
|---|---|---|
| Hot (local TSDB) | 15 days | SSD-backed persistent volume |
| Warm (Thanos) | 90 days | S3 / GCS object storage |
| Cold (archive) | 1 year+ | S3 Glacier / GCS Nearline |

Storage sizing is based on an estimated ingestion rate of ~500K samples/second across all security exporters. Local TSDB is capped at 200 GB with overflow automatically tiered to object storage by Thanos.

---

## Access Control & RBAC

Grafana access is governed by SSO via Okta SAML. Users are assigned to roles based on their team membership in Okta groups. No local Grafana accounts are permitted in production.

| Role | Access Level |
|---|---|
| security-engineer | All dashboards, read-only Prometheus API |
| sec-ops-analyst | Cybersec dashboards, alert acknowledgement |
| platform-admin | Full admin access, alert rule management |
| executive | Overview and compliance dashboards only |

Prometheus's own HTTP API is accessible only within the cluster network. External access is routed exclusively through Grafana.

---

## Runbooks

Runbooks for every critical and high severity alert are maintained in the internal wiki and linked directly from each alert's annotations.

**Wiki root:** `https://wiki.internal/runbooks/prometheus-cybersec/`

| Alert | Runbook |
|---|---|
| EDR Agent Offline | https://wiki.internal/runbooks/edr-agent-offline |
| Vault Sealed | https://wiki.internal/runbooks/vault-sealed |
| Critical Vuln Spike | https://wiki.internal/runbooks/critical-vuln-spike |
| MFA Failure Surge | https://wiki.internal/runbooks/mfa-failure-surge |
| WAF Block Rate Anomaly | https://wiki.internal/runbooks/waf-block-rate |
| SIEM Ingestion Lag | https://wiki.internal/runbooks/siem-ingestion-lag |

---

## Contributing

1. Create a feature branch from `main` using the prefix `feat/`, `fix/`, or `chore/`
2. Follow the exporter naming convention documented in `docs/exporter-conventions.md`
3. All new alert rules must include a `severity` label, a `team` label, a suppression duration, and a `runbook` annotation
4. Validate your Prometheus config and alert rules locally using `promtool` before opening a pull request
5. Pull requests require **2 approvals from Security Engineers** and **1 approval from a Platform Engineer** before merge

---

## License

This project is licensed under the [MIT License](LICENSE).  
Internal tooling, exporter credentials, and target configurations are **not included** in this repository and are managed separately via the secrets management pipeline.

---

> **Security Notice:** If you discover a vulnerability in this observability stack, please report it via our [responsible disclosure process](https://github.com/20MH1A04H9/Prometheus/issues) — do **not** open a public GitHub issue.
