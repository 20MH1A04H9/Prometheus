# 🚀 Prometheus Installation on Windows Server

![Windows](https://img.shields.io/badge/Platform-Windows-blue)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange)
![Grafana](https://img.shields.io/badge/Visualization-Grafana-red)
![Status](https://img.shields.io/badge/Setup-Guide-green)

A complete guide to installing **Prometheus on Windows Server** for infrastructure monitoring.

---

## 📌 Table of Contents

- Overview
- Architecture
- Download Prometheus
- Extract Prometheus
- Configure Prometheus
- Run Prometheus
- Access Web UI
- Install as Windows Service
- Verify Prometheus
- Add Windows Exporter
- Useful Queries
- Default Ports
- Recommended Architecture
- Summary

---

# 📖 Overview

**Prometheus** is an open-source monitoring and alerting system used to collect metrics from:

- Servers  
- Applications  
- Network Devices  
- Cloud Infrastructure  

Prometheus stores metrics as **time-series data** and supports querying through:

- PromQL  
- Grafana dashboards  
- Alertmanager integrations  

Although Prometheus is commonly installed on Linux, it can also run on **Windows Server**.

---

# 🏗 Prometheus Architecture

```text
Servers / Applications
        │
        │ Metrics
        ▼
Exporters (Node Exporter / Windows Exporter)
        │
        ▼
Prometheus Server
        │
        ▼
Alertmanager
        │
        ▼
Grafana Dashboards
```

---

# 📥 Download Prometheus

Download Prometheus from the official release page:

[Prometheus Releases](https://github.com/prometheus/prometheus/releases?utm_source=chatgpt.com)

Download the Windows package:

```bash
prometheus.windows-amd64.zip
```

Example:

```bash
prometheus-3.9.1.windows-amd64.zip
```

---

# 📂 Extract Prometheus

Copy the ZIP file to your Windows server.

Example location:

```bash
C:\Prometheus
```

Extract the package.

### Folder Structure

```bash
C:\Prometheus
├── prometheus.exe
├── promtool.exe
├── prometheus.yml
├── consoles
└── console_libraries
```

---

# ⚙ Configure Prometheus

Open:

```bash
prometheus.yml
```

Example configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"

    static_configs:
      - targets: ["localhost:9090"]
```

This configuration allows Prometheus to monitor itself.

---

# ▶ Run Prometheus on Windows

Open **PowerShell** or **Command Prompt**

```powershell
cd C:\Prometheus
```

Start Prometheus:

```powershell
prometheus.exe --config.file=prometheus.yml
```

Prometheus runs on:

```bash
9090
```

---

# 🌐 Access Prometheus Web Interface

Open your browser:

```bash
http://localhost:9090
```

You should see the Prometheus dashboard.

---

# 🔧 Run Prometheus as Windows Service (Recommended)

Prometheus can be installed as a Windows service using **NSSM**.

## What is NSSM?

**Non-Sucking Service Manager (NSSM)** helps run applications as Windows services.

Download NSSM:

[NSSM Download](https://nssm.cc/download)

---

## Extract NSSM

Create directory:

```bash
C:\nssm
```

Move `nssm.exe` from the extracted package to this folder.

---

## Create Prometheus Service

Open **Terminal as Administrator**

```powershell
cd C:\nssm
```

Run:

```powershell
.\nssm.exe install Prometheus
```

Configure:

| Field | Value |
|--------|---------|
| Path | C:\Prometheus\prometheus.exe |
| Startup Directory | C:\Prometheus |
| Arguments | --config.file=C:\Prometheus\prometheus.yml |

Click **Install Service**

---

## Start Service

```powershell
net start Prometheus
```

Check services:

```powershell
services.msc
```

---

# ✅ Verify Prometheus

Open:

```bash
http://SERVER-IP:9090
```

Navigate to:

```bash
Status → Targets
```

Expected output:

```bash
prometheus → UP
```

---

# 🖥 Add Windows Monitoring

Install :contentReference[oaicite:2]{index=2} for monitoring Windows systems.

Default Port:

```bash
9182
```

Example configuration:

```yaml
scrape_configs:
  - job_name: "windows_servers"

    static_configs:
      - targets:
        - "192.168.1.10:9182"
```

---

# 📊 Useful PromQL Queries

### CPU Usage

```promql
rate(process_cpu_seconds_total[5m])
```

### Memory Usage

```promql
process_resident_memory_bytes
```

### System Uptime

```promql
process_start_time_seconds
```

---

# 🔌 Default Ports

| Service | Port |
|----------|--------|
| Prometheus | 9090 |
| Windows Exporter | 9182 |
| Grafana | 3000 |

---

# 🏢 Recommended Monitoring Stack

```text
Windows / Linux Servers
        │
    Exporters
        │
    Prometheus Server
        │
    Alertmanager
        │
    Grafana Dashboards
```

---

# ✅ Summary

Prometheus can run directly on Windows Server for infrastructure monitoring.

When integrated with:

- :contentReference[oaicite:3]{index=3}  
- :contentReference[oaicite:4]{index=4}  
- :contentReference[oaicite:5]{index=5}  

…it becomes a complete monitoring solution for servers and applications.