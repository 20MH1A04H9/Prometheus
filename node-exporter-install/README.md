<!-- PROMETHEUS NODE EXPORTER SETUP GUIDE -->
<!-- ██████╗ ██╗   ██╗    ██╗   ██╗██╗███████╗██╗    ██╗ █████╗  -->
<!-- ██╔══██╗╚██╗ ██╔╝    ██║   ██║██║██╔════╝██║    ██║██╔══██╗ -->
<!-- ██████╔╝ ╚████╔╝     ██║   ██║██║███████╗██║ █╗ ██║███████║ -->
<!-- ██╔══██╗  ╚██╔╝      ╚██╗ ██╔╝██║╚════██║██║███╗██║██╔══██║ -->
<!-- ██████╔╝   ██║        ╚████╔╝ ██║███████║╚███╔███╔╝██║  ██║ -->
<!-- ╚═════╝    ╚═╝         ╚═══╝  ╚═╝╚══════╝ ╚══╝╚══╝ ╚═╝  ╚═╝ -->

<div align="center">

```
██████╗ ██████╗  ██████╗ ███╗   ███╗███████╗████████╗██╗  ██╗███████╗██╗   ██╗███████╗
██╔══██╗██╔══██╗██╔═══██╗████╗ ████║██╔════╝╚══██╔══╝██║  ██║██╔════╝██║   ██║██╔════╝
██████╔╝██████╔╝██║   ██║██╔████╔██║█████╗     ██║   ███████║█████╗  ██║   ██║███████╗
██╔═══╝ ██╔══██╗██║   ██║██║╚██╔╝██║██╔══╝     ██║   ██╔══██║██╔══╝  ██║   ██║╚════██║
██║     ██║  ██║╚██████╔╝██║ ╚═╝ ██║███████╗   ██║   ██║  ██║███████╗╚██████╔╝███████║
╚═╝     ╚═╝  ╚═╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝   ╚═╝   ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚══════╝
                    NODE EXPORTER // SETUP GUIDE // v1.8.1
```

<br/>

[![Prometheus](https://img.shields.io/badge/◈_PROMETHEUS-E6522C?style=for-the-badge&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Node Exporter](https://img.shields.io/badge/◈_NODE_EXPORTER-00FF41?style=for-the-badge&logo=linux&logoColor=black)](https://github.com/prometheus/node_exporter)
[![Linux](https://img.shields.io/badge/◈_LINUX-1793D1?style=for-the-badge&logo=linux&logoColor=white)](https://www.linux.org/)
[![Grafana](https://img.shields.io/badge/◈_GRAFANA-F46800?style=for-the-badge&logo=grafana&logoColor=white)](https://grafana.com/)
[![Status](https://img.shields.io/badge/◈_STATUS-OPERATIONAL-00FF41?style=for-the-badge)](/)
[![Author](https://img.shields.io/badge/◈_ART_BY-VISWA-blueviolet?style=for-the-badge)](/)

<br/>

```
┌─────────────────────────────────────────────────────────────────────┐
│  [SYS] Initializing monitoring stack...                             │
│  [OK]  Node Exporter agent deployed on port 9100                   │
│  [OK]  Prometheus scrape target registered                          │
│  [OK]  Grafana dashboard connected                                  │
│  [>>>] All systems NOMINAL — metrics pipeline ACTIVE               │
└─────────────────────────────────────────────────────────────────────┘
```

</div>

---

## ◈ TABLE OF CONTENTS

```
[01] ─── OVERVIEW
[02] ─── ARCHITECTURE & DATA FLOW
[03] ─── INSTALL NODE EXPORTER
         ├── [STEP-01] Create System User
         ├── [STEP-02] Download Binary
         ├── [STEP-03] Extract Files
         └── [STEP-04] Install Binary
[04] ─── CREATE SYSTEMD SERVICE
[05] ─── START & ENABLE SERVICE
[06] ─── VERIFY METRICS ENDPOINT
[07] ─── CONFIGURE PROMETHEUS SCRAPE
[08] ─── RESTART & VALIDATE PROMETHEUS
[09] ─── TEST PROMQL QUERIES
[10] ─── SUMMARY
```

---

## [01] ◈ OVERVIEW

> **Node Exporter** is an official Prometheus exporter that exposes hardware and OS-level metrics from Linux systems. It acts as a local telemetry agent, transmitting real-time system data to your Prometheus server.

Metrics are served over HTTP at:

```bash
http://<SERVER-IP>:9100/metrics
```

### ◈ METRICS COLLECTED

| Vector | Description |
|---|---|
| `node_cpu_seconds_total` | CPU utilization across all cores |
| `node_memory_MemAvailable_bytes` | Available memory in bytes |
| `node_filesystem_size_bytes` | Disk capacity per mount point |
| `node_network_receive_bytes_total` | Inbound network traffic |
| `node_load1` | 1-minute system load average |
| `node_disk_io_time_seconds_total` | Disk I/O time spent |

---

## [02] ◈ ARCHITECTURE & DATA FLOW

```
╔══════════════════════════════════════════════════════════╗
║                   MONITORING STACK                       ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║   ┌──────────────────────────────────────────────────┐  ║
║   │   LINUX SERVER(S) — Target Hosts                 │  ║
║   │   ┌─────────────────────────────────────────┐   │  ║
║   │   │  NODE EXPORTER  ░░ :9100/metrics        │   │  ║
║   │   └────────────────────┬────────────────────┘   │  ║
║   └────────────────────────┼───────────────────────-┘  ║
║                            │  HTTP PULL (scrape)        ║
║                            ▼                            ║
║   ┌──────────────────────────────────────────────────┐  ║
║   │   PROMETHEUS SERVER  ░░ :9090                    │  ║
║   │   Time-Series Database & Query Engine            │  ║
║   └────────────────────────┬─────────────────────────┘  ║
║                            │                            ║
║              ┌─────────────┴──────────┐                 ║
║              ▼                        ▼                 ║
║   ┌──────────────────┐   ┌────────────────────────┐    ║
║   │  ALERTMANAGER    │   │   GRAFANA DASHBOARD    │    ║
║   │  :9093           │   │   :3000                │    ║
║   └──────────────────┘   └────────────────────────┘    ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

---

## [03] ◈ INSTALL NODE EXPORTER

> Official releases: [github.com/prometheus/node_exporter/releases](https://github.com/prometheus/node_exporter/releases)

### ▸ STEP-01 — Create Dedicated System User

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

> **[WHY]** Running Node Exporter under a dedicated unprivileged user follows the principle of least privilege — a core security hardening practice.

---

### ▸ STEP-02 — Download Node Exporter Binary

```bash
cd /tmp

wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```

---

### ▸ STEP-03 — Extract Archive

```bash
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz

cd node_exporter-1.8.1.linux-amd64
```

---

### ▸ STEP-04 — Install Binary & Set Permissions

```bash
sudo cp node_exporter /usr/local/bin/

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

**Verify installation:**

```bash
node_exporter --version
```

Expected output:
```
node_exporter, version 1.8.1 (branch: HEAD, ...)
```

---

## [04] ◈ CREATE SYSTEMD SERVICE

Create the service unit file:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste the following configuration:

```ini
[Unit]
Description=Prometheus Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

# Auto-restart on failure
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

---

## [05] ◈ START & ENABLE SERVICE

```bash
# Reload systemd to register the new service
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable node_exporter

# Start the service
sudo systemctl start node_exporter

# Verify service is running
sudo systemctl status node_exporter
```

Expected output:
```
● node_exporter.service - Prometheus Node Exporter
   Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled)
   Active: active (running) since ...
```

---

## [06] ◈ VERIFY METRICS ENDPOINT

**From the server (CLI):**

```bash
curl http://localhost:9100/metrics
```

**From browser:**

```
http://<SERVER-IP>:9100/metrics
```

You should see a stream of metrics like:

```bash
# HELP node_cpu_seconds_total Seconds the CPUs spent in each mode
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 12345.67

node_memory_MemAvailable_bytes 4294967296
node_filesystem_size_bytes{...} 107374182400
node_network_receive_bytes_total{device="eth0"} 892341
```

> **[SYS]** If the endpoint is unreachable, check firewall rules: `sudo ufw allow 9100/tcp`

---

## [07] ◈ CONFIGURE PROMETHEUS SCRAPE TARGET

Edit the Prometheus config:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

### Single Server

```yaml
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["192.168.x.x:9100"]
```

### Multiple Servers

```yaml
scrape_configs:
  - job_name: "linux_servers"
    static_configs:
      - targets:
          - "192.168.1.10:9100"   # web-server-01
          - "192.168.1.11:9100"   # db-server-01
          - "192.168.1.12:9100"   # app-server-01
```

> **[TIP]** Use labels to categorize your targets for easier filtering in Grafana:
> ```yaml
>         labels:
>           env: "production"
>           role: "webserver"
> ```

---

## [08] ◈ RESTART & VALIDATE PROMETHEUS

```bash
# Restart Prometheus to apply new config
sudo systemctl restart prometheus

# Confirm service is healthy
sudo systemctl status prometheus
```

**Validate in Prometheus UI:**

```
http://<SERVER-IP>:9090
```

Navigate to:

```
Status  ──►  Targets
```

Expected state:

```
┌─────────────────────────────────────────────────────┐
│  Job: node_exporter          State: ● UP            │
│  Endpoint: http://192.168.x.x:9100/metrics          │
│  Last Scrape: 0.123s ago     Labels: {...}           │
└─────────────────────────────────────────────────────┘
```

---

## [09] ◈ TEST PROMQL QUERIES

Open Prometheus UI → **Graph** tab and run:

```promql
# CPU usage percentage per core
100 - (avg by(cpu) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Available memory in GB
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024

# Disk usage percentage
(node_filesystem_size_bytes - node_filesystem_free_bytes) / node_filesystem_size_bytes * 100

# Network receive rate (bytes/sec)
rate(node_network_receive_bytes_total[5m])

# System load average (1 min)
node_load1
```

---

## [10] ◈ SUMMARY

```
╔══════════════════════════════════════════════════════╗
║              DEPLOYMENT CHECKLIST                    ║
╠══════════════════════════════════════════════════════╣
║  [✔] node_exporter user created (least privilege)   ║
║  [✔] Binary downloaded & installed to /usr/local/   ║
║  [✔] Ownership set to node_exporter:node_exporter   ║
║  [✔] Systemd service created & enabled              ║
║  [✔] Service started & verified active              ║
║  [✔] Metrics endpoint accessible on :9100           ║
║  [✔] Prometheus scrape target configured            ║
║  [✔] Prometheus restarted & target showing UP       ║
║  [✔] PromQL queries validated in UI                 ║
╚══════════════════════════════════════════════════════╝
```

> **[SYS]** Monitoring pipeline fully operational. All telemetry agents nominal.

---

<div align="center">

```
┌────────────────────────────────────────────┐
│                                            │
│    ◈  Crafted with precision by Viswa  ◈   │
│    ◈  Prometheus • Node Exporter Guide ◈   │
│                                            │
└────────────────────────────────────────────┘
```

[![Star this repo](https://img.shields.io/badge/⭐_STAR_THIS_REPO-If_it_helped_you-FFD700?style=for-the-badge)](/)

</div>
