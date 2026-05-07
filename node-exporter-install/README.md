# 🚀 Prometheus Node Exporter Setup Guide  
### *Art by Viswa* ✨

![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange)
![Node Exporter](https://img.shields.io/badge/Exporter-Node%20Exporter-green)
![Linux](https://img.shields.io/badge/Platform-Linux-blue)
![Status](https://img.shields.io/badge/Setup-Guide-success)

A complete guide to installing **Prometheus Node Exporter** on Linux servers and integrating it with :contentReference[oaicite:0]{index=0} and :contentReference[oaicite:1]{index=1}.

---

## 📌 Table of Contents

- Overview  
- Architecture Flow  
- Install Node Exporter  
- Create Systemd Service  
- Start Node Exporter  
- Verify Metrics  
- Configure Prometheus  
- Restart Prometheus  
- Verify in Prometheus UI  
- Test Queries  
- Summary  

---

# 📖 Overview


One of the most commonly used exporters is:


It exposes Linux hardware and operating system metrics through:

```bash
http://<SERVER-IP>:9100/metrics
```

### Metrics Collected

- CPU Usage  
- Memory Usage  
- Disk Usage  
- Network Traffic  
- File System Statistics  
- System Load  

---

# 🏗 Architecture Flow

```text
Linux Server
     │
     │ Metrics
     ▼
Node Exporter (Port 9100)
     │
     │ HTTP Pull
     ▼
Prometheus Server (Port 9090)
     │
     ▼
Grafana Dashboard
```

---

# ⚙ Install Node Exporter

Download official release:

[Node Exporter Releases](https://github.com/prometheus/node_exporter/releases?utm_source=chatgpt.com)

---

## Step 1: Create User

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

---

## Step 2: Download Node Exporter

```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```

---

## Step 3: Extract Files

```bash
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz
cd node_exporter-1.8.1.linux-amd64
```

---

## Step 4: Install Binary

```bash
sudo cp node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Verify installation:

```bash
node_exporter --version
```

---

# 🔧 Create Systemd Service

Create service file:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

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

Restart=always

[Install]
WantedBy=multi-user.target
```

---

# ▶ Start Node Exporter

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable service:

```bash
sudo systemctl enable node_exporter
```

Start service:

```bash
sudo systemctl start node_exporter
```

Check status:

```bash
sudo systemctl status node_exporter
```

---

# ✅ Verify Node Exporter Locally

```bash
curl http://localhost:9100/metrics
```

You should see metrics like:

```bash
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_size_bytes
node_network_receive_bytes_total
```

---

# 🌐 Verify from Browser

Open:

```bash
http://SERVER-IP:9100/metrics
```

---

# ⚙ Configure Prometheus Scrape Target

Edit configuration:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add:

```yaml
scrape_configs:
  - job_name: "node_exporter"

    static_configs:
      - targets: ["192.168.x.x:9100"]
```

---

## Multiple Servers Example

```yaml
scrape_configs:
  - job_name: "linux_servers"

    static_configs:
      - targets:
        - "192.168.x.x:9100"
        - "192.168.x.x:9100"
        - "192.168.x.x:9100"
```

---

# 🔄 Restart Prometheus

```bash
sudo systemctl restart prometheus
```

Check status:

```bash
sudo systemctl status prometheus
```

---

# 📊 Verify in Prometheus UI

Open:

```bash
http://SERVER-IP:9090
```

Navigate:

```bash
Status → Targets
```

Expected result:

```bash
node_exporter → UP
```

---

# 📈 Test PromQL Query

Run this query in Prometheus UI:

```promql
node_cpu_seconds_total
```

Architecture Flow:

```text
Linux Servers
     │
Node Exporter (9100)
     │
Prometheus Server
     │
Alertmanager
     │
Grafana Dashboard
```

---

### ⭐ Art by Viswa