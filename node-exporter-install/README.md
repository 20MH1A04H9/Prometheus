# 🖥️ Node Exporter Install

> Deploys the Prometheus Node Exporter on Linux-based security infrastructure hosts to expose system-level hardware and OS metrics for observability.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Collected Metrics](#collected-metrics)
- [Security Hosts Covered](#security-hosts-covered)
- [Alerting](#alerting)
- [Grafana Dashboard](#grafana-dashboard)

---

## Overview

Node Exporter runs as a **systemd service** on every Linux host that supports the cybersecurity infrastructure. It exposes kernel and hardware-level metrics that Prometheus scrapes to monitor host health, detect resource exhaustion, and trigger alerts before security tools are impacted.

This folder contains the installation playbooks, systemd unit configuration, and scrape configuration for all managed Linux security hosts.

---

## Folder Structure

```
node-exporter-install/
├── ansible/
│   ├── install-node-exporter.yml    # Ansible playbook for deployment
│   └── inventory/
│       ├── siem-hosts               # SIEM server inventory
│       ├── vault-hosts              # HashiCorp Vault nodes
│       └── scanner-hosts            # Vulnerability scanner hosts
├── systemd/
│   └── node_exporter.service        # Systemd unit file
├── tls/
│   └── tls-config.yml               # Node Exporter TLS configuration
├── prometheus-scrape.yml            # Scrape config snippet
└── README.md
```

---

## Configuration Files

### `systemd/node_exporter.service`

Runs Node Exporter as a dedicated `node_exporter` system user with minimal privileges. The service is set to restart automatically on failure and starts on every boot.

Enabled collectors in this deployment:

| Collector | What It Measures |
|---|---|
| `cpu` | CPU utilization per core |
| `diskstats` | Disk I/O throughput and latency |
| `filesystem` | Disk space and inode usage |
| `meminfo` | Memory usage, swap, cache |
| `netdev` | Network interface traffic |
| `systemd` | Systemd unit states |
| `processes` | Running process count and states |
| `interrupts` | Hardware interrupt rates |

### `tls/tls-config.yml`

Node Exporter is configured to serve its metrics endpoint over **HTTPS with mutual TLS**, so that only the Prometheus server can scrape it. Certificates are issued by the internal CA and managed by cert-manager.

---

## Collected Metrics

Key metrics used in security observability alerting:

| Metric | Alert Condition |
|---|---|
| `node_cpu_seconds_total` | CPU saturation > 90% for 10 min |
| `node_memory_MemAvailable_bytes` | Available memory < 10% |
| `node_filesystem_avail_bytes` | Disk free < 15% on any partition |
| `node_disk_io_time_seconds_total` | I/O utilization > 95% |
| `node_network_receive_drop_total` | Packet drops increasing |
| `node_systemd_unit_state` | Critical service unit not `active` |
| `node_load1` | 1-minute load > CPU count × 2 |

---

## Security Hosts Covered

Node Exporter is deployed to all Linux hosts running security infrastructure components:

| Host Group | Role |
|---|---|
| SIEM Nodes | Splunk indexers, Elastic data nodes |
| Vault Cluster | HashiCorp Vault primary and secondary |
| Vulnerability Scanners | Tenable Nessus, Qualys scanner appliances |
| Log Aggregators | Rsyslog, Fluentd, Logstash forwarders |
| Security Jump Hosts | Bastion / PAM-managed access hosts |
| Falco Nodes | Container runtime security agents |

---

## Alerting

Alert rules referencing Node Exporter metrics are located in the main Prometheus rules folder under `rules/node-alerts.yml`.

| Alert | Condition | Severity |
|---|---|---|
| `HostHighCPU` | CPU > 90% for 10 min | high |
| `HostLowMemory` | Available memory < 10% | high |
| `HostDiskFull` | Disk free < 15% | critical |
| `HostDown` | Node Exporter unreachable | critical |
| `HostServiceCrashed` | Systemd unit in `failed` state | critical |
| `HostHighLoad` | Load average > 2× CPU count | medium |

---

## Grafana Dashboard

The **Node Exporter Full** dashboard (Grafana ID: 1860) is provisioned and pre-filtered by security host group. It provides:

- Per-host CPU, memory, disk, and network utilization
- Systemd service health at a glance
- I/O saturation heatmaps
- Network traffic anomaly detection (sudden spikes)

---

> **Security Note:** The Node Exporter metrics endpoint must never be exposed to the internet. Firewall rules restrict port `9100` to the Prometheus server IP only, enforced by both OS-level `iptables` and Kubernetes NetworkPolicy where applicable.
