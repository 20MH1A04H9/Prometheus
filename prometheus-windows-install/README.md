# 🪟 Prometheus Windows Install

> Deploys Prometheus on Windows Server for environments where the primary security monitoring infrastructure runs on Windows, including AD-integrated SIEM nodes and Windows-based PAM servers.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Windows-Specific Scrape Targets](#windows-specific-scrape-targets)
- [Alerting](#alerting)
- [Service Management](#service-management)
- [Security Hardening](#security-hardening)

---

## Overview

This folder covers the deployment of **Prometheus as a Windows Service** on Windows Server hosts used in the cybersecurity stack. It supplements the Kubernetes-based Prometheus deployment for Windows-only environments — primarily Active Directory, CyberArk PAM, and Windows-based SIEM components — where containerized deployment is not feasible.

Prometheus on Windows scrapes:
- The **Windows Exporter** (running locally or on target hosts)
- Windows-specific security exporters (Active Directory, CyberArk)
- Any HTTP-based exporter reachable from the Windows host

---

## Folder Structure

```
prometheus-windows-install/
├── prometheus.yml             # Windows-specific Prometheus configuration
├── rules/
│   ├── windows-alerts.yml     # Windows host alert rules
│   └── ad-alerts.yml          # Active Directory specific alerts
├── nssm/
│   └── install-service.md     # Steps to register Prometheus as a Windows service via NSSM
├── tls/
│   └── tls-config.md          # TLS setup for Windows (cert store integration)
└── README.md
```

---

## Configuration Files

### `prometheus.yml`

Configured for the Windows environment with these adaptations:

- Storage path set to `C:\prometheus\data`
- Log format set to `json` for compatibility with Windows Event Forwarder
- Scrape targets reference Windows Exporter and AD Exporter on localhost and domain-joined hosts
- Remote write configured to send metrics to the central Thanos Receiver over HTTPS

### `rules/windows-alerts.yml`

Alert rules specific to Windows infrastructure. Covers Windows service states, event log error rates, and Active Directory health.

### `rules/ad-alerts.yml`

Dedicated rules for Active Directory observability, covering replication lag, failed authentication rates, and domain controller availability.

---

## Windows-Specific Scrape Targets

| Target | Exporter | Port | Metrics |
|---|---|---|---|
| Windows AD servers | windows-exporter | 9182 | CPU, memory, disk, services |
| Active Directory | ad-exporter | 9166 | Replication, auth failures, DC health |
| CyberArk PAM | cyberark-exporter | 9190 | Session counts, vault connectivity, failures |
| IIS (if applicable) | windows-exporter | 9182 | Request rates, errors, response times |
| Windows Defender | windows-exporter | 9182 | Threat detection events, update status |

---

## Alerting

| Alert | Condition | Severity |
|---|---|---|
| `WindowsServiceDown` | Critical Windows service not running | critical |
| `ADReplicationFailing` | AD replication lag > 60 minutes | critical |
| `ADAuthFailureSurge` | Failed logins > 100 in 5 minutes | high |
| `CyberArkVaultUnreachable` | PAM vault connection failing | critical |
| `WindowsDiskFull` | Disk free < 15% on any volume | critical |
| `WindowsHighCPU` | CPU > 90% for 10 minutes | high |
| `DefenderThreatDetected` | Defender threat event count increasing | high |

---

## Service Management

Prometheus runs as a **Windows Service** registered via NSSM (Non-Sucking Service Manager). This ensures it:

- Starts automatically on Windows boot
- Restarts on crash with configurable delay
- Logs stdout/stderr to the Windows Event Log
- Runs under a dedicated low-privilege service account (`svc-prometheus`)

Service management commands are documented in `nssm/install-service.md`.

---

## Security Hardening

- Prometheus runs under a dedicated `svc-prometheus` account with no interactive login rights
- The metrics HTTP port is bound to `localhost` only — not exposed on network interfaces
- Windows Firewall rules restrict access to the storage and config directories to the service account only
- Remote write to Thanos uses a client certificate issued by the internal PKI
- Prometheus config directory ACLs are set to deny write access to all accounts except `svc-prometheus` and Domain Admins

---

> **Active Directory Integration:** Prometheus on Windows is domain-joined, allowing it to use Kerberos-based authentication when scraping AD Exporter endpoints on other domain controllers.
