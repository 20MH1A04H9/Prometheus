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

# ✅ Prerequisites


## Grafana Setup Options

You can install  using  any of these methods:

- [Grafana Local Setup](https://guides.hakedev.com/wiki/windows/grafana?utm_source=chatgpt.com)
- [Grafana Cloud Setup](https://guides.hakedev.com/wiki/general/grafana-cloud?utm_source=chatgpt.com)
- [Grafana Docker Setup](https://guides.hakedev.com/wiki/general/grafana-docker?utm_source=chatgpt.com)

---

## Prometheus Setup

Install  as a Windows service:

- [Prometheus Windows Service Setup](https://guides.hakedev.com/wiki/windows/prometheus-service?utm_source=chatgpt.com)

---

# 📥 Download Windows Exporter

## Download Latest Version

Download the latest release from the official GitHub repository:

[Windows Exporter Releases](https://github.com/prometheus-community/windows_exporter/releases?utm_source=chatgpt.com)

Example package:

```bash
windows_exporter-0.29.2-amd64.msi
```

This `.msi` installer will be used for the setup process.
---

# 📥 Download Windows Exporter

Download the latest release from:

[Windows Exporter Releases](https://github.com/prometheus-community/windows_exporter/releases?utm_source=chatgpt.com)

Example package:

```bash
windows_exporter-0.29.2-amd64.msi
```

---

# ⚙ Install Windows Exporter

Run the `.msi` installer.

If Windows shows a security warning:

- Click **More Info**
- Click **Run Anyway**

Grant administrator permission.

---

## Firewall Option

If Prometheus runs on another server:

✅ Enable firewall exception during installation.

This allows Prometheus to scrape metrics remotely.

---

# 📂 Installation Path

Default installation directory:

```bash
C:\Program Files\windows_exporter
```

---

# 🔧 Update Prometheus Configuration

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

If exporter runs on another machine:

```yaml
- targets: ["SERVER-IP:9182"]
```

---

# 🚨 Fix Config Issues

## Step 1: Delete Wrong Config Folder

Delete:

```bash
C:\Program Files\windows_exporter\config
```

There should be **no config folder**.

---

## Step 2: Rename Config File

Rename:

```bash
config.yaml
```

To:

```bash
config.yml
```

Final location:

```bash
C:\Program Files\windows_exporter\config.yml
```

---

# Advance Config File for the Windows

```yaml
collectors:
  enabled:
    # System metrics
    - cpu
    - memory
    - logical_disk
    - net
    - system
    - os
    - process
    - service
    
    # Additional hardware collectors
    - physical_disk
    - diskdrive
    - mscluster_cluster
    - mscluster_network
    - mscluster_node
    - mscluster_resource
    - mscluster_resourcegroup
    
    # Network collectors
    - tcp
    - udp
    - netframework_clrexceptions
    - netframework_clrinterop
    - netframework_clrjit
    - netframework_clrloading
    - netframework_clrlocksandthreads
    - netframework_clrmemory
    - netframework_clrremoting
    - netframework_clrsecurity
    
    # Windows specific collectors
    - ad
    - adcs
    - adfs
    - cache
    - container
    - cs
    - dfsr
    - dhcp
    - dns
    - exchange
    - fsrmquota
    - hyperv
    - iis
    - license
    - logon
    - msmq
    - mssql
    - netframework_clrexceptions
    - netframework_clrinterop
    - netframework_clrjit
    - netframework_clrloading
    - netframework_clrlocksandthreads
    - netframework_clrmemory
    - netframework_clrremoting
    - netframework_clrsecurity
    - nps
    - printer
    - remote_fx
    - scheduled_task
    - smb
    - smbclient
    - smtp
    - textfile
    - thermalzone
    - time
    - update
    - vmware
    - vmware_blast

# Service collector configuration
# Monitor specific Windows services
collector:
  service:
    services-where: "Name='wuauserv' OR Name='MSSQLSERVER' OR Name='Spooler' OR Name='W32Time' OR Name='WinRM' OR Name='bits' OR Name='EventLog' OR Name='Schedule' OR Name='Dnscache' OR Name='LanmanServer' OR Name='LanmanWorkstation' OR Name='RpcSs' OR Name='SamSs' OR Name='CryptSvc' OR Name='TrustedInstaller' OR Name='Dhcp' OR Name='Netlogon' OR Name='Netman' OR Name='NlaSvc' OR Name='nsi' OR Name='WinHttpAutoProxySvc' OR Name='Winmgmt' OR Name='WSearch' OR Name='SENS' OR Name='ShellHWDetection' OR Name='Themes' OR Name='AudioSrv' OR Name='AudioEndpointBuilder' OR Name='FontCache' OR Name='PlugPlay' OR Name='BFE' OR Name='mpssvc' OR Name='wscsvc' OR Name='WdNisSvc' OR Name='WinDefend' OR Name='SysMain' OR Name='UsoSvc' OR Name='VSS' OR Name='VaultSvc' OR Name='Wecsvc' OR Name='WerSvc' OR Name='wuauserv' OR Name='lmhosts' OR Name='SessionEnv' OR Name='TermService' OR Name='UmRdpService' OR Name='Browser' OR Name='MSDTC' OR Name='p2pimsvc' OR Name='p2psvc' OR Name='PNRPsvc' OR Name='RemoteRegistry' OR Name='seclogon' OR Name='SSDPSRV' OR Name='upnphost' OR Name='DiagTrack' OR Name='DPS' OR Name='WdiServiceHost' OR Name='WdiSystemHost' OR Name='gpsvc' OR Name='iphlpsvc' OR Name='ProfSvc' OR Name='stisvc' OR Name='tiledatamodelsvc' OR Name='TokenBroker' OR Name='WbioSrvc' OR Name='KeyIso' OR Name='AppIDSvc' OR Name='IKEEXT' OR Name='Appinfo' OR Name='AppReadiness' OR Name='AppXSvc' OR Name='ClipSVC' OR Name='CoreMessagingRegistrar' OR Name='DcomLaunch' OR Name='EFS' OR Name='EventSystem' OR Name='LSM' OR Name='MMCSS' OR Name='NgcCtnrSvc' OR Name='NgcSvc' OR Name='Power' OR Name='PcaSvc' OR Name='SCardSvr' OR Name='ScDeviceEnum' OR Name='SCPolicySvc' OR Name='SENS' OR Name='StateRepository' OR Name='StorSvc' OR Name='SystemEventsBroker' OR Name='TabletInputService' OR Name='TextInputManagementService' OR Name='TimeBrokerSvc' OR Name='UserManager' OR Name='WpnService' OR Name='CDPUserSvc' OR Name='OneSyncSvc' OR Name='UnistoreSvc' OR Name='WpnUserService'"

# Process collector configuration
collector:
  process:
    # Whitelist specific processes to monitor
    include: ".*"
    # Blacklist certain processes
    exclude: ""

# Logical disk collector configuration
collector:
  logical_disk:
    # Include volumes - blank = all volumes
    include: ""
    # Exclude volumes by drive letter or mount point
    exclude: ""

# Network interface collector configuration
collector:
  net:
    # Include interfaces - blank = all interfaces
    include: ".*"
    # Exclude specific interfaces
    exclude: ""

# Physical disk collector configuration
collector:
  physical_disk:
    # Include disks
    include: ".*"
    # Exclude specific disks
    exclude: ""

# IIS collector configuration (if IIS is installed)
collector:
  iis:
    # Site whitelist regex
    site-include: ".*"
    site-exclude: ""
    # App whitelist regex
    app-include: ".*"
    app-exclude: ""

# MSSQL collector configuration (if SQL Server is installed)
collector:
  mssql:
    # SQL Server instances to monitor
    enabled-classes: "accessmethods,availreplica,bufman,databases,dbreplica,genstats,locks,memmgr,sqlstats,sqlerrors,transactions"

# Scheduled Task collector configuration
collector:
  scheduled_task:
    # Tasks to include
    include: ".*"
    # Tasks to exclude
    exclude: ""

# Text file collector configuration
collector:
  textfile:
    # Directory containing text files with metrics
    directory: "C:\\custom_metrics"

# Printer collector configuration
collector:
  printer:
    # Include printers
    include: ".*"
    exclude: ""

# License collector configuration
collector:
  license:
    # Nothing to configure

# Thermalzone collector configuration
collector:
  thermalzone:
    # Nothing to configure

# Time collector configuration
collector:
  time:
    # Nothing to configure

# Global exporter configuration
log:
  level: "info"
  format: "logfmt"

web:
  listen-address: ":9182"
  telemetry-path: "/metrics"

# Timeout configurations
scrape:
  timeout-margin: 0.5

# Memory limits (optional)
runtime:
  gomaxprocs: 1

```

---

# Advanced Config

Enable additional collectors like:

- IIS  
- MSSQL  
- DHCP  
- DNS  
- Hyper-V  
- VMware  
- Printer  
- Scheduled Tasks  

---

# 🔄 Restart Windows Exporter

Run Command Prompt as Administrator:

```bash
net stop windows_exporter
net start windows_exporter
```

---

# ✅ Verify Exporter

Open browser:

```bash
http://localhost:9182/metrics
```

Search for:

```bash
windows_cpu_time_total
windows_memory_available_bytes
windows_service_status
```

If visible → exporter is working.

---

# 🌐 Multi-Server Monitoring

## Example Architecture

| Machine | Role | IP | Port |
|----------|--------|------------|------|
| VM-1 | Prometheus Server | 10.2.0.230 | 9090 |
| VM-2 | Windows Server | 10.2.1.18 | 9182 |
| VM-3 | Windows Server | 10.2.1.19 | 9182 |
| VM-4 | Windows Server | 10.2.1.20 | 9182 |

---

# Install Exporter on All Windows Servers

```bash
windows_exporter.exe --listen-address=":9182"
```

Verify:

```bash
http://localhost:9182/metrics
```

Remote verification:

```bash
http://WINDOWS-IP:9182/metrics
```

---

# Open Firewall Port

Run as Administrator:

```bash
netsh advfirewall firewall add rule name="Windows Exporter 9182" dir=in action=allow protocol=TCP localport=9182
```

---

# Configure Prometheus for Multiple Servers

Edit:

```bash
prometheus.yml
```

Add:

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

# Verify in Prometheus UI

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

# 📊 Grafana Integration

Your existing :contentReference[oaicite:6]{index=6} dashboards will automatically display all Windows servers.

Use dashboard filters:

- instance  
- hostname  
- job  

---

# 🏗 Final Architecture

```text
Windows Server 1 → Windows Exporter
Windows Server 2 → Windows Exporter
Windows Server 3 → Windows Exporter
           │
           ▼
      Prometheus Server
           │
           ▼
        Grafana
```

---

# ✅ Summary

