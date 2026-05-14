# 🔭 Blackbox Exporter — Website & Network Monitoring

![Linux](https://img.shields.io/badge/Platform-Linux-blue)
![Prometheus](https://img.shields.io/badge/Monitoring-Prometheus-orange)
![Grafana](https://img.shields.io/badge/Visualization-Grafana-red)
![Blackbox](https://img.shields.io/badge/Exporter-Blackbox-purple)
![Status](https://img.shields.io/badge/Setup-Guide-green)

> Prometheus-based external probing for HTTP, ICMP, DNS, and TCP targets — with a Grafana dashboard for real-time visibility into uptime, SSL expiry, response times, and availability.

---

## 📐 Architecture

```
Prometheus (10.0.0.10)
    └── asks Blackbox Exporter (10.0.0.10:9115)
            └── probes target websites / IPs
                    └── returns metrics back to Prometheus
                            └── visualized in Grafana
```

| Server | Role | Components |
|--------|------|------------|
| `10.0.0.10` (monitoring-server) | Monitoring server | Prometheus + Blackbox Exporter + Grafana |
| `10.0.0.20` (app-server) | Monitored target | Nothing required |

---

## 🚀 Installation

### Step 1 — Download & Install Blackbox Exporter

```bash
cd /tmp
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
tar -xvf blackbox_exporter-0.25.0.linux-amd64.tar.gz
sudo mv blackbox_exporter-0.25.0.linux-amd64/blackbox_exporter /usr/local/bin/
chmod +x /usr/local/bin/blackbox_exporter

# Verify
blackbox_exporter --version
```

---

### Step 2 — Configure Blackbox Modules

```bash
sudo mkdir -p /etc/blackbox_exporter
sudo nano /etc/blackbox_exporter/blackbox.yml
```

```yaml
modules:
  # HTTP 2xx check
  http_2xx:
    prober: http
    timeout: 10s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []  # Defaults to 2xx
      method: GET
      follow_redirects: true
      preferred_ip_protocol: "ip4"
      tls_config:
        insecure_skip_verify: false

  # HTTPS with TLS info
  http_2xx_tls:
    prober: http
    timeout: 10s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []
      method: GET
      follow_redirects: true
      preferred_ip_protocol: "ip4"
      tls_config:
        insecure_skip_verify: false

  # ICMP ping
  icmp:
    prober: icmp
    timeout: 5s
    icmp:
      preferred_ip_protocol: "ip4"

  # DNS lookup
  dns_udp:
    prober: dns
    timeout: 5s
    dns:
      query_name: "google.com"
      transport_protocol: "udp"

  # TCP connect
  tcp_connect:
    prober: tcp
    timeout: 5s
```

---

### Step 3 — Create systemd Service

```bash
sudo useradd --no-create-home --shell /bin/false blackbox

sudo nano /etc/systemd/system/blackbox_exporter.service
```

```ini
[Unit]
Description=Prometheus Blackbox Exporter
After=network.target

[Service]
User=blackbox
Group=blackbox
ExecStart=/usr/local/bin/blackbox_exporter \
    --config.file=/etc/blackbox_exporter/blackbox.yml \
    --web.listen-address=:9115
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable blackbox_exporter
sudo systemctl start blackbox_exporter
sudo systemctl status blackbox_exporter

# Verify metrics endpoint
curl http://localhost:9115/metrics | head -10

# Test probing a website
curl "http://localhost:9115/probe?target=https://example.com&module=http_2xx"
```

> **Grant ICMP capability** (required for ping probes):
> ```bash
> sudo setcap cap_net_raw+ep /usr/local/bin/blackbox_exporter
> getcap /usr/local/bin/blackbox_exporter
> # Expected: /usr/local/bin/blackbox_exporter = cap_net_raw+ep
> ```

---

### Step 4 — Add Scrape Jobs to Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add the following under `scrape_configs:`:

```yaml
  # ── Blackbox HTTP monitoring ──────────────────────────────────
  - job_name: 'blackbox_http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://grafana.example.com
          - http://10.0.0.20
          - https://app.example.com
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.0.0.10:9115

  # ── Blackbox ICMP ping monitoring ─────────────────────────────
  - job_name: 'blackbox_icmp'
    metrics_path: /probe
    params:
      module: [icmp]
    static_configs:
      - targets:
          - 10.0.0.10
          - 10.0.0.20
          - 10.0.0.30
          - 8.8.8.8
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.0.0.10:9115

  # ── Blackbox DNS monitoring ────────────────────────────────────
  - job_name: 'blackbox_dns'
    metrics_path: /probe
    params:
      module: [dns_udp]
    static_configs:
      - targets:
          - 8.8.8.8
          - 1.1.1.1
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.0.0.10:9115
```

```bash
# Validate config
promtool check config /etc/prometheus/prometheus.yml

# Restart Prometheus
sudo systemctl restart prometheus
sudo systemctl status prometheus

# Verify blackbox targets are discovered
curl -s "http://localhost:9090/api/v1/targets" | python3 -m json.tool | grep -E "blackbox|job"
```

---

### Step 5 — Import Grafana Dashboard

1. Go to **Grafana → Dashboards → Import**
2. Enter Dashboard ID: **`7587`** *(Prometheus Blackbox Exporter)*
3. Click **Load**
4. Set datasource → **Prometheus**
5. Configure variables:
   - `job` for HTTP → `blackbox_http`
   - `job` for ICMP → `blackbox_icmp`
6. Click **Import**

> Alternatively, import the bundled `blackbox-dashboard.json` via **Dashboards → Import → Upload JSON file**.

---

### Step 6 — Verify Probing

```bash
curl -s "http://localhost:9115/probe?target=https://example.com&module=http_2xx" \
  | grep -E "probe_success|probe_http_status_code|probe_tls_version_info|probe_ssl_earliest_cert_expiry"
```

Expected output:
```
probe_success 1
probe_http_status_code 200
probe_tls_version_info{version="TLS 1.3"} 1
probe_ssl_earliest_cert_expiry 1.7XXXXXXXXX
```

---

## 📊 Dashboard Panels

| Panel | Metric |
|-------|--------|
| ✅ HTTP Status Code | `probe_http_status_code` |
| 🔒 TLS Version | `probe_tls_version_info` |
| 📜 Certificate Expiry | `probe_ssl_earliest_cert_expiry` |
| ⏱ DNS Lookup Time | `probe_dns_lookup_time_seconds` |
| 🌐 HTTP Version | `probe_http_version` |
| 📡 ICMP Ping | `probe_icmp_duration_seconds` |
| 📊 24h / 3d / 7d Availability | `avg_over_time(probe_success)` |
| ⏳ Probe Duration | `probe_duration_seconds` |

### Dashboard Sections

| Section | Panels |
|---------|--------|
| 🌐 Status Overview | Probe Status, HTTP Code, SSL Expiry, Duration, HTTP Version, SSL Status |
| 📊 Availability | Last 24h / 3 Days / 7 Days % uptime |
| ⏱ Probe Timings | Duration over time, HTTP phase breakdown (Connect / TLS / Processing / Transfer) |
| 🔒 SSL/TLS | Certificate expiry countdown + bar gauge for all targets |
| 🌍 DNS | Lookup duration over time + average |
| 📡 ICMP | Ping duration + reachability status |
| 📋 History | Probe success history + HTTP status code history |

---

## 🛠 Troubleshooting

### Blackbox targets show as `down`

```bash
sudo systemctl status blackbox_exporter
ss -tlnp | grep 9115
curl http://localhost:9115/metrics | head -5
```

### ICMP probe_success = 0 (Azure / Cloud IPs)

Azure and many cloud providers block ICMP at the firewall level. This is expected behavior — not an exporter issue.

Test with a known-pingable host:
```bash
curl -s "http://localhost:9115/probe?target=8.8.8.8&module=icmp" | grep probe_success
```

If ICMP works on public IPs but not your server IPs, ensure the binary has the required capability:
```bash
sudo setcap cap_net_raw+ep /usr/local/bin/blackbox_exporter
getcap /usr/local/bin/blackbox_exporter
# Expected: cap_net_raw+ep
```

---

## 📦 Stack

| Component | Version | Port |
|-----------|---------|------|
| Prometheus | latest | 9090 |
| Blackbox Exporter | v0.25.0 | 9115 |
| Grafana | latest | 3000 |

---

## 📁 Files

```
.
├── blackbox.yml              # Blackbox probe module config
├── prometheus.yml            # Prometheus scrape config (blackbox jobs)
├── blackbox_exporter.service # systemd unit file
└── blackbox-dashboard.json   # Grafana dashboard (import-ready)
```

---

*Part of a Prometheus-based SOC monitoring infrastructure.*