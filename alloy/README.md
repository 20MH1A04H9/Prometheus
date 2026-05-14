# 📡 Grafana Alloy — Windows Log & Metrics Forwarding

![Windows](https://img.shields.io/badge/Platform-Windows-blue)
![Grafana Alloy](https://img.shields.io/badge/Agent-Grafana%20Alloy-orange)
![Loki](https://img.shields.io/badge/Logs-Loki-yellow)
![Prometheus](https://img.shields.io/badge/Metrics-Prometheus-red)
![Status](https://img.shields.io/badge/Setup-Guide-green)

> Centralised log and metrics collection from Windows 11 endpoints using **Grafana Alloy**, forwarding Windows Event Logs, PowerShell transcripts, SonicWall VPN logs, and system metrics to a remote Loki + Prometheus stack.

---

## 📐 Architecture

```
┌─────────────────────────────────────────────┐
│            Loki / Grafana Server            │
│                                             │
│  Grafana Alloy Config Server (port 8080)   │
│  → stores ONE master config.alloy          │
│  → all endpoints pull from it              │
└─────────────┬───────────────────────────────┘
              │  pulls config every 5 min
    ┌─────────┼─────────┐
    ▼         ▼         ▼
Endpoint1  Endpoint2  Endpoint3 ... (10+)
(Alloy)    (Alloy)    (Alloy)
```

| Component | Role | Address |
|-----------|------|---------|
| Loki | Log aggregation | `http://monitoring-server:3100` |
| Prometheus | Metrics storage | `http://monitoring-server:9090` |
| Grafana | Visualisation | `http://monitoring-server:3000` |
| Alloy (per endpoint) | Log + metrics agent | runs on each Windows host |

---

## 📦 What Gets Collected

### 🗂 Windows Event Logs

| Source | Job Label | Description |
|--------|-----------|-------------|
| `System` | `windows-system` | OS-level system events |
| `Application` | `windows-application` | Application errors and warnings |
| `Security` | `windows-security` | Logon/logoff, privilege use, auditing |
| `Windows PowerShell` | `windows-powershell` | PowerShell engine events |
| `Microsoft-Windows-PowerShell/Operational` | `powershell-commands` | Script block logging (executed commands) |
| `Microsoft-Windows-Windows Defender/Operational` | `windows-defender` | AV detections and scans |
| `Microsoft-Windows-TaskScheduler/Operational` | `windows-tasks` | Scheduled task activity |
| `Microsoft-Windows-Windows Firewall.../Firewall` | `windows-firewall` | Firewall rule hits |

### 🖥 RDP Monitoring

| Source | Job Label | `log_type` |
|--------|-----------|------------|
| `TerminalServices-LocalSessionManager/Operational` | `rdp-monitoring` | `session` |
| `TerminalServices-RemoteConnectionManager/Operational` | `rdp-monitoring` | `connection` |
| `TerminalServices-RDPClient/Operational` | `rdp-monitoring` | `client` |
| `RemoteDesktopServices-RdpCoreTS/Operational` | `rdp-monitoring` | `core` |

### 📁 File-Based Logs

| Source | Job Label | Path |
|--------|-----------|------|
| PowerShell Transcripts | `powershell-transcripts` | `C:\PSLogs\**\*.txt` |
| SonicWall NetExtender | `sonicwall-netextender` | `NetExtender.dbg`, `WxacLog.dbg` |

### 📊 Metrics

| Exporter | Job Label | Port |
|----------|-----------|------|
| Windows Exporter | `windows-exporter` | `9182` |

---

## ⚙️ Alloy Configuration

### Full `config.alloy`

```hcl
// ─────────────────────────────────────────────────────
// POWERSHELL TRANSCRIPTS - C:\PSLogs\**\*.txt
// ─────────────────────────────────────────────────────
local.file_match "ps_transcripts" {
  path_targets = [
    {
      __path__ = "C:\\PSLogs\\**\\*.txt",
      job      = "powershell-transcripts",
      host     = constants.hostname,
      os       = "windows11",
    },
  ]
}

loki.source.file "ps_transcripts_scrape" {
  targets    = local.file_match.ps_transcripts.targets
  forward_to = [loki.write.remote_loki.receiver]
}

// ─────────────────────────────────────────────────────
// WINDOWS EVENT LOGS
// ─────────────────────────────────────────────────────
loki.source.windowsevent "system" {
  eventlog_name          = "System"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-system",
    host = constants.hostname,
    os   = "windows11",
    env  = "production",
  }
}

loki.source.windowsevent "application" {
  eventlog_name          = "Application"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-application",
    host = constants.hostname,
    os   = "windows11",
    env  = "production",
  }
}

loki.source.windowsevent "security" {
  eventlog_name          = "Security"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-security",
    host = constants.hostname,
    os   = "windows11",
    env  = "production",
  }
}

loki.source.windowsevent "powershell" {
  eventlog_name          = "Windows PowerShell"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-powershell",
    host = constants.hostname,
    os   = "windows11",
  }
}

loki.source.windowsevent "ps_scriptblock" {
  eventlog_name          = "Microsoft-Windows-PowerShell/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "powershell-commands",
    host = constants.hostname,
    os   = "windows11",
  }
}

loki.source.windowsevent "defender" {
  eventlog_name          = "Microsoft-Windows-Windows Defender/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-defender",
    host = constants.hostname,
    os   = "windows11",
  }
}

loki.source.windowsevent "task_scheduler" {
  eventlog_name          = "Microsoft-Windows-TaskScheduler/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-tasks",
    host = constants.hostname,
    os   = "windows11",
  }
}

loki.source.windowsevent "firewall" {
  eventlog_name          = "Microsoft-Windows-Windows Firewall With Advanced Security/Firewall"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job  = "windows-firewall",
    host = constants.hostname,
    os   = "windows11",
  }
}

// ─────────────────────────────────────────────────────
// RDP MONITORING
// ─────────────────────────────────────────────────────
loki.source.windowsevent "rdp_session" {
  eventlog_name          = "Microsoft-Windows-TerminalServices-LocalSessionManager/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job      = "rdp-monitoring",
    host     = constants.hostname,
    os       = "windows11",
    log_type = "session",
  }
}

loki.source.windowsevent "rdp_connection" {
  eventlog_name          = "Microsoft-Windows-TerminalServices-RemoteConnectionManager/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job      = "rdp-monitoring",
    host     = constants.hostname,
    os       = "windows11",
    log_type = "connection",
  }
}

loki.source.windowsevent "rdp_client" {
  eventlog_name          = "Microsoft-Windows-TerminalServices-RDPClient/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job      = "rdp-monitoring",
    host     = constants.hostname,
    os       = "windows11",
    log_type = "client",
  }
}

loki.source.windowsevent "rdp_core" {
  eventlog_name          = "Microsoft-Windows-RemoteDesktopServices-RdpCoreTS/Operational"
  use_incoming_timestamp = true
  forward_to             = [loki.write.remote_loki.receiver]
  labels = {
    job      = "rdp-monitoring",
    host     = constants.hostname,
    os       = "windows11",
    log_type = "core",
  }
}

// ─────────────────────────────────────────────────────
// SONICWALL VPN LOGS
// ─────────────────────────────────────────────────────
local.file_match "sonicwall_logs" {
  path_targets = [
    {
      __path__ = "C:\\Program Files (x86)\\SonicWall\\SSL-VPN\\NetExtender\\NetExtender.dbg",
      job      = "sonicwall-netextender",
      host     = constants.hostname,
      app      = "sonicwall",
    },
    {
      __path__ = "C:\\Program Files (x86)\\SonicWall\\SSL-VPN\\NetExtender\\WxacLog.dbg",
      job      = "sonicwall-netextender",
      host     = constants.hostname,
      app      = "sonicwall",
    },
  ]
}

loki.source.file "sonicwall_scrape" {
  targets    = local.file_match.sonicwall_logs.targets
  forward_to = [loki.write.remote_loki.receiver]
}

// ─────────────────────────────────────────────────────
// WINDOWS METRICS - AUTO IP DETECTION VIA REGEX
// ─────────────────────────────────────────────────────
prometheus.scrape "windows_exporter" {
  targets = [{
    __address__ = "localhost:9182",
  }]
  forward_to      = [prometheus.relabel.fix_instance.receiver]
  scrape_interval = "15s"
  job_name        = "windows-exporter"
}

prometheus.relabel "fix_instance" {
  forward_to = [prometheus.remote_write.metrics_server.receiver]

  // Step 1: Copy hostname into instance label
  rule {
    target_label = "instance"
    replacement  = constants.hostname
  }

  // Step 2: Set host label from hostname
  rule {
    target_label = "host"
    replacement  = constants.hostname
  }

  // Step 3: Extract IP from __address__ using regex
  rule {
    source_labels = ["__address__"]
    regex         = "localhost(.*)"
    target_label  = "ip_port"
    replacement   = "${constants.hostname}${1}"
  }

  // Step 4: Set final instance as hostname:port
  rule {
    source_labels = ["__address__"]
    regex         = "(.*):([0-9]+)"
    target_label  = "instance"
    replacement   = "${constants.hostname}:9182"
  }
}

// ─────────────────────────────────────────────────────
// REMOTE WRITE — PROMETHEUS
// ─────────────────────────────────────────────────────
prometheus.remote_write "metrics_server" {
  endpoint {
    url = "http://monitoring-server:9090/api/v1/write"
  }
  external_labels = {
    host = constants.hostname,
    os   = "windows11",
    env  = "production",
  }
}

// ─────────────────────────────────────────────────────
// REMOTE WRITE — LOKI
// ─────────────────────────────────────────────────────
loki.write "remote_loki" {
  endpoint {
    url               = "http://monitoring-server:3100/loki/api/v1/push"
    retry_on_http_429 = true
  }
  external_labels = {
    server = constants.hostname,
    os     = "windows11",
    env    = "production",
  }
}
```

---

## 🔧 Updating Endpoints (URL Change)

Only 2 lines need updating when the monitoring server address changes:

| Setting | Config Key | Example |
|---------|------------|---------|
| Prometheus remote write | `prometheus.remote_write > endpoint > url` | `http://monitoring-server:9090/api/v1/write` |
| Loki remote write | `loki.write > endpoint > url` | `http://monitoring-server:3100/loki/api/v1/push` |

After updating `config.alloy`, restart Alloy on each Windows endpoint:

```powershell
Restart-Service -Name "Alloy"
```

---

## 🚀 Deployment

### Prerequisites

- [Grafana Alloy](https://grafana.com/docs/alloy/latest/) installed on each Windows endpoint
- Windows Exporter running on port `9182`
- PowerShell transcript logging enabled (`C:\PSLogs\`)
- SonicWall NetExtender installed (if monitoring VPN logs)

### Install Alloy (Windows)

```powershell
# Download and install Alloy MSI from Grafana releases
# https://github.com/grafana/alloy/releases

# After install, place config.alloy at:
# C:\Program Files\GrafanaLabs\Alloy\config.alloy

# Start the service
Start-Service -Name "Alloy"

# Verify running
Get-Service -Name "Alloy"
```

### Configure Remote Pull (Optional)

To have all endpoints pull config from a central server instead of managing config files individually, expose `config.alloy` via HTTP on port `8080` from the monitoring server and point each Alloy agent to it.

---

## 🛠 Troubleshooting

### Alloy service won't start

```powershell
# Check service status
Get-Service -Name "Alloy"

# View logs
Get-EventLog -LogName Application -Source "Alloy" -Newest 20
```

### Logs not appearing in Loki

```powershell
# Test Loki connectivity from endpoint
Invoke-WebRequest -Uri "http://monitoring-server:3100/ready"

# Verify Alloy config syntax (run from Alloy install dir)
alloy fmt config.alloy
```

### Metrics not appearing in Prometheus

```powershell
# Verify Windows Exporter is running
Invoke-WebRequest -Uri "http://localhost:9182/metrics"

# Check Prometheus remote write endpoint
Invoke-WebRequest -Uri "http://monitoring-server:9090/-/ready"
```

---

## 📦 Stack

| Component | Role | Default Port |
|-----------|------|-------------|
| Grafana Alloy | Log + metrics agent (Windows) | — |
| Windows Exporter | Windows metrics | `9182` |
| Loki | Log aggregation | `3100` |
| Prometheus | Metrics storage | `9090` |
| Grafana | Visualisation | `3000` |

---

