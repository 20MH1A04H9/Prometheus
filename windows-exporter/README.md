# 🪟 Windows Exporter Setup

> Deploys and configures the Prometheus Windows Exporter on Windows Server hosts running cybersecurity infrastructure, exposing OS-level and application metrics for Prometheus scraping.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Enabled Collectors](#enabled-collectors)
- [Security Host Coverage](#security-host-coverage)
- [Collected Metrics](#collected-metrics)
- [Alerting](#alerting)
- [Grafana Dashboard](#grafana-dashboard)
- [Security Hardening](#security-hardening)

---

## Overview

The Windows Exporter is the Windows equivalent of Node Exporter. It runs as a **Windows Service** on each monitored host and exposes hundreds of Windows-native metrics via an HTTP endpoint for Prometheus to scrape.

In this deployment, Windows Exporter is the primary source of host health metrics for:

- **Active Directory Domain Controllers**
- **CyberArk PAM Vault servers**
- **Windows-based SIEM components**
- **Windows jump hosts / bastion servers**
- **Microsoft Defender-managed endpoints**

---

## Folder Structure

```
windows-exporter/
├── config.yml                      # Windows Exporter collector configuration
├── ansible/
│   └── deploy-windows-exporter.yml # Ansible WinRM playbook for mass deployment
├── prometheus-scrape.yml           # Scrape config snippet for prometheus.yml
├── alerts/
│   └── windows-exporter-alerts.yml # Alert rules for Windows host metrics
├── tls/
│   └── tls-setup.md                # HTTPS + mTLS configuration for Windows
└── README.md
```

---

## Configuration Files

### `config.yml` — Collector Selection

Only the collectors relevant to cybersecurity infrastructure monitoring are enabled. Unused collectors are explicitly disabled to reduce attack surface and metrics cardinality.

### `prometheus-scrape.yml`

A ready-to-paste scrape config block for `prometheus.yml`. Includes file-based service discovery so new Windows hosts can be added by updating the targets file without reloading Prometheus.

---

## Enabled Collectors

| Collector | Metrics Provided | Why Enabled |
|---|---|---|
| `cpu` | Per-core utilization | Detect resource exhaustion on security hosts |
| `memory` | RAM usage, page faults | Identify memory pressure on SIEM/PAM nodes |
| `logical_disk` | Disk space, I/O rates | Alert on full log or data volumes |
| `net` | Network throughput, errors | Detect unusual traffic on security hosts |
| `service` | Windows service states | Alert on crashed security services |
| `process` | Per-process CPU/memory | Identify rogue or runaway processes |
| `os` | Uptime, user sessions | Track unauthorized sessions |
| `system` | System calls, threads | Baseline system activity |
| `ad` | AD object counts, errors | Domain controller health |
| `dns` | DNS query rates, failures | DNS anomaly detection |
| `iis` | Request rates, errors | If IIS hosts security web apps |
| `mssql` | Query latency, connections | SIEM database monitoring |
| `scheduled_task` | Task run states | Detect tampered scheduled tasks |
| `textfile` | Custom metrics from files | Inject custom security tool metrics |

Disabled collectors: `exchange`, `hyperv`, `print`, `terminal_services` (not used in this environment).

---

## Security Host Coverage

| Host Group | Key Collectors Active |
|---|---|
| Active Directory DCs | `ad`, `dns`, `service`, `cpu`, `memory` |
| CyberArk PAM Vault | `service`, `process`, `logical_disk`, `net` |
| Windows SIEM Nodes | `cpu`, `memory`, `logical_disk`, `mssql` |
| Bastion / Jump Hosts | `os`, `service`, `net`, `scheduled_task` |
| Defender Endpoints | `service`, `process`, `scheduled_task` |

---

## Collected Metrics

Key metrics mapped to security observability use cases:

| Metric | Security Use Case |
|---|---|
| `windows_service_state` | Alert when CyberArk, Defender, or AD services fail |
| `windows_ad_replication_pending_operations` | Detect AD replication stalls |
| `windows_net_packets_received_discarded_total` | Network anomaly detection |
| `windows_logical_disk_free_megabytes` | Prevent log volume exhaustion on SIEM nodes |
| `windows_os_logged_in_users` | Alert on unexpected active sessions |
| `windows_process_cpu_time_total` | Detect runaway processes on security hosts |
| `windows_scheduled_task_last_result` | Alert on failed or modified scheduled tasks |
| `windows_dns_total_query_received_total` | DNS query volume anomaly |

---

## Alerting

Alert rules are in `alerts/windows-exporter-alerts.yml` and are loaded by the central Prometheus rule engine.

| Alert | Condition | Severity |
|---|---|---|
| `WindowsCriticalServiceDown` | Security-critical service not in `running` state | critical |
| `WindowsDiskSpaceCritical` | Free disk < 15% on any volume | critical |
| `WindowsHighMemory` | Available memory < 10% | high |
| `WindowsADReplicationStalled` | Pending AD replication ops > 0 for 60 min | critical |
| `WindowsUnexpectedLogon` | Active logged-in users on bastion outside business hours | high |
| `WindowsScheduledTaskFailed` | Scheduled task last result ≠ 0 for security-related tasks | high |
| `WindowsDefenderServiceDown` | Windows Defender service not running | critical |

---

## Grafana Dashboard

Import `windows-exporter-dashboard.json` (or use Grafana dashboard ID **14694**) for the Windows overview. Security-specific panels added on top:

- **Service health grid** — all security-critical Windows services at a glance
- **Active sessions tracker** — logged-in users per host over time
- **AD replication health** — pending operations and last sync time
- **Disk space forecast** — projected time to full for SIEM data volumes
- **Process anomaly panel** — processes with unusual CPU or memory compared to baseline

---

## Security Hardening

- Windows Exporter runs under a dedicated `svc-winexporter` service account with no interactive login and read-only access to WMI
- The metrics port (`9182`) is bound to localhost only; Prometheus accesses it over a TLS-secured reverse proxy or direct scrape with NetworkPolicy enforcement
- Windows Firewall inbound rules permit port `9182` only from the Prometheus server IP
- Collector configuration disables any collector that exposes sensitive process arguments or environment variables

---

> **Deployment Scale:** Windows Exporter is deployed via Ansible WinRM to all domain-joined Windows security hosts. The target inventory is maintained in `ansible/inventory/` and grouped by security function.
