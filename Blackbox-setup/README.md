# 🔍 Blackbox Exporter Setup

> Probes external endpoints (HTTP, HTTPS, TCP, ICMP, DNS) and exposes results as Prometheus metrics. Used for uptime monitoring of cybersecurity tools and APIs.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Probe Types](#probe-types)
- [Monitored Targets](#monitored-targets)
- [Alerting](#alerting)
- [Grafana Dashboard](#grafana-dashboard)

---

## Overview

The Blackbox Exporter runs active probes against external and internal security tool endpoints. It is the primary source of **availability and latency metrics** for all cybersecurity SaaS integrations and internal APIs.

---

## Folder Structure

```
Blackbox-setup/
├── blackbox.yml              # Probe module definitions
├── prometheus-scrape.yml     # Scrape config snippet to add to prometheus.yml
├── alerts/
│   └── blackbox-alerts.yml   # Alerting rules for probe failures
└── README.md
```

---

## Configuration Files

### `blackbox.yml` — Probe Modules

Defines the probe types used across security tool monitoring.

| Module | Protocol | Use Case |
|---|---|---|
| `http_2xx` | HTTP/HTTPS | Confirm endpoint returns 200–299 |
| `http_post_2xx` | HTTPS POST | API health check with payload |
| `tcp_connect` | TCP | Port reachability (SIEM, EDR APIs) |
| `icmp_probe` | ICMP | Host-level reachability |
| `dns_lookup` | DNS | Validate security domain resolution |
| `tls_expiry` | HTTPS | Certificate expiry within 30 days |

---

### `prometheus-scrape.yml` — Scrape Integration

Add this block to your main `prometheus.yml` under `scrape_configs` to activate Blackbox probing.

Targets are defined per security domain:

| Target Group | Examples |
|---|---|
| SIEM endpoints | Splunk HEC, Elastic API, Sentinel ingestion |
| EDR cloud APIs | CrowdStrike API, SentinelOne management |
| Identity providers | Okta, Azure AD token endpoints |
| Vault health | HashiCorp Vault `/v1/sys/health` |
| WAF management | Cloudflare API, AWS WAF health |

---

## Probe Types

### HTTP/HTTPS Probes

Used to validate that security tool APIs and dashboards are reachable and returning expected status codes. TLS validation is enforced — probes will fail if a certificate is invalid or expired.

### TCP Probes

Used for raw port connectivity checks to SIEM ingestors, EDR agents, and network security appliances where HTTP probing is not applicable.

### TLS Certificate Expiry

A dedicated module tracks days remaining on TLS certificates for all monitored security endpoints. Alerts fire at 30 days and 7 days before expiry.

### DNS Probes

Validates that security-critical domain names resolve correctly, catching DNS hijacking or misconfigurations before they affect tool connectivity.

---

## Monitored Targets

| Security Tool | Probe Module | Expected Outcome |
|---|---|---|
| CrowdStrike Falcon API | `http_2xx` | 200 OK |
| Splunk HEC Endpoint | `http_post_2xx` | 200 OK with ack |
| HashiCorp Vault | `http_2xx` | `initialized: true` |
| Okta SSO | `http_2xx` | 200 OK |
| Palo Alto NGFW Mgmt | `tcp_connect` | Port 443 open |
| Qualys Scanner | `http_2xx` | 200 OK |
| Internal PKI / CA | `tls_expiry` | > 30 days remaining |

---

## Alerting

Alert rules are located in `alerts/blackbox-alerts.yml`.

| Alert | Condition | Severity |
|---|---|---|
| `EndpointDown` | Probe fails for > 2 minutes | critical |
| `SSLCertExpiringSoon` | Certificate expires in < 30 days | high |
| `SSLCertExpiryCritical` | Certificate expires in < 7 days | critical |
| `SlowProbeLatency` | HTTP duration > 3s for > 5 min | medium |
| `DNSResolutionFailing` | DNS probe fails for > 2 minutes | high |

All alerts include a `runbook` annotation pointing to `https://wiki.internal/runbooks/blackbox/`.

---

## Grafana Dashboard

Import `blackbox-dashboard.json` into Grafana. The dashboard provides:

- **Probe success rate** per target (heat map)
- **HTTP response latency** trends
- **TLS certificate expiry countdown** for all endpoints
- **Probe failure timeline** with annotations for incidents
- **Uptime SLA** percentage per security tool (30-day rolling)

---

> **Note:** Blackbox probes run every 30 seconds. For high-sensitivity endpoints (Vault, identity providers), interval is reduced to 15 seconds via a dedicated scrape job.
