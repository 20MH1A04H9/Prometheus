<div align="center">

# 🖥️ Windows Exporter Setup Guide

<br/>

[![Windows Exporter](https://img.shields.io/badge/◈_WINDOWS_EXPORTER-0078D4?style=for-the-badge&logo=windows&logoColor=white)](https://github.com/prometheus-community/windows_exporter)
[![Prometheus](https://img.shields.io/badge/◈_PROMETHEUS-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/◈_GRAFANA-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)
[![Windows](https://img.shields.io/badge/◈_WINDOWS_SERVER-0078D4?style=for-the-badge&logo=windows&logoColor=white)](https://www.microsoft.com/windows)
[![Status](https://img.shields.io/badge/◈_STATUS-OPERATIONAL-00FF41?style=for-the-badge)](/)
[![Author](https://img.shields.io/badge/◈_AUTHOR-VISWA-blueviolet?style=for-the-badge)](/)

<br/>

</div>

---

# 📖 Introduction

This guide explains how to install **Windows Exporter** and integrate it with:

- Prometheus
- Grafana

It enables Windows infrastructure monitoring using Prometheus metrics.

---

# ✅ Prerequisites

## Grafana Setup Options

- [Grafana Local Setup](https://guides.hakedev.com/wiki/windows/grafana)
- [Grafana Cloud Setup](https://guides.hakedev.com/wiki/general/grafana-cloud)
- [Grafana Docker Setup](https://guides.hakedev.com/wiki/general/grafana-docker)

## Prometheus Setup

- [Prometheus Windows Service Setup](https://guides.hakedev.com/wiki/windows/prometheus-service)

---

# 📥 Download Windows Exporter

Download latest release:

https://github.com/prometheus-community/windows_exporter/releases

Example:

```bash
windows_exporter-0.29.2-amd64.msi
```

---

# ⚙ Install Windows Exporter

Run the MSI installer.

If prompted:

- Click **More Info**
- Click **Run Anyway**
- Allow administrator permissions

---

# Firewall Configuration

If Prometheus runs on another server:

Enable firewall exception during installation.

---

# Installation Path

```bash
C:\Program Files\windows_exporter
```

---

# Configure Prometheus

Edit:

```bash
prometheus.yml
```

Add:

```yaml
- job_name: "windows_exporter"
  scrape_interval: 15s
  static_configs:
    - targets: ["localhost:9182"]
```

For remote host:

```yaml
- targets: ["SERVER-IP:9182"]
```

---

# Fix Config Issues

### Delete incorrect folder

```bash
C:\Program Files\windows_exporter\config
```

### Rename config file

```bash
config.yaml → config.yml
```

Final path:

```bash
C:\Program Files\windows_exporter\config.yml
```

---

# Restart Windows Exporter

```bash
net stop windows_exporter
net start windows_exporter
```

---

# Verify Exporter

Open:

```bash
http://localhost:9182/metrics
```

Check for:

- windows_cpu_time_total
- windows_memory_available_bytes
- windows_service_status

---

# Multi Server Monitoring

| Machine | Role | IP | Port |
|----------|--------|------------|------|
| VM-1 | Prometheus | 10.2.0.230 | 9090 |
| VM-2 | Windows | 10.2.1.18 | 9182 |
| VM-3 | Windows | 10.2.1.19 | 9182 |
| VM-4 | Windows | 10.2.1.20 | 9182 |

---

# Open Firewall Port

```bash
netsh advfirewall firewall add rule name="Windows Exporter 9182" dir=in action=allow protocol=TCP localport=9182
```

---

# Prometheus Multi Target Config

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  - job_name: "windows_exporter"
    static_configs:
      - targets:
          - "10.2.1.18:9182"
          - "10.2.1.19:9182"
          - "10.2.1.20:9182"
```

---

# Restart Prometheus

```bash
net stop prometheus
net start prometheus
```

---

# Verify Prometheus Targets

Open:

```bash
http://10.2.0.230:9090/targets
```

Expected:

| Job | Instance | Status |
|------|------------|----------|
| windows_exporter | 10.2.1.18:9182 | UP |
| windows_exporter | 10.2.1.19:9182 | UP |
| windows_exporter | 10.2.1.20:9182 | UP |

---

# Grafana Integration

Your Grafana dashboards will automatically show all Windows servers.

Filters:

- instance
- hostname
- job

---

# Architecture

```text
Windows Server → Windows Exporter
Windows Server → Windows Exporter
Windows Server → Windows Exporter
        ↓
   Prometheus Server
        ↓
      Grafana
```

---

# ✅ Summary

Windows Exporter enables Prometheus to monitor Windows infrastructure metrics including:

- CPU
- Memory
- Disk
- Services
- Network
- Processes

Combined with Grafana, it provides full Windows monitoring.