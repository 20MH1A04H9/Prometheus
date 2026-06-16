# 🔗 Grafana Alloy Setup

> Grafana Alloy is the OpenTelemetry-native collector used to collect, transform, and forward metrics, logs, and traces from cybersecurity tools to Prometheus and Grafana Loki.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Folder Structure](#folder-structure)
- [Configuration Files](#configuration-files)
- [Collection Pipelines](#collection-pipelines)
- [Security Tool Integrations](#security-tool-integrations)
- [Forwarding Targets](#forwarding-targets)
- [Service Discovery](#service-discovery)

---

## Overview

Grafana Alloy (formerly Grafana Agent) replaces traditional single-purpose collectors with a unified, programmable pipeline. In this deployment, Alloy handles:

- **Metrics collection** from security tools that do not support native Prometheus scraping
- **Log forwarding** from security event sources to Grafana Loki
- **Trace ingestion** from API gateways and WAF layers
- **Remote write** to Prometheus-compatible endpoints including Thanos

---

## Folder Structure

```
alloy/
├── config.alloy               # Main Alloy pipeline configuration (River syntax)
├── components/
│   ├── siem-logs.alloy        # SIEM log collection pipeline
│   ├── edr-metrics.alloy      # EDR metric scrape and relabelling
│   ├── vault-telemetry.alloy  # Vault audit log + metrics pipeline
│   └── waf-logs.alloy         # WAF request log ingestion
├── secrets/
│   └── README.md              # How secrets are injected (Vault Agent Sidecar)
└── README.md
```

---

## Configuration Files

### `config.alloy` — Main Pipeline

Written in Alloy's River configuration language. Defines all component blocks that make up the collection, processing, and forwarding pipelines.

Key component types used:

| Component Type | Purpose |
|---|---|
| `prometheus.scrape` | Scrape metrics from security exporters |
| `prometheus.relabel` | Relabel metrics before forwarding |
| `prometheus.remote_write` | Push metrics to Thanos / Mimir |
| `loki.source.file` | Tail log files from security agents |
| `loki.write` | Forward logs to Grafana Loki |
| `otelcol.receiver.otlp` | Receive OpenTelemetry traces |
| `discovery.kubernetes` | Auto-discover exporter pods in Kubernetes |

---

## Collection Pipelines

### Metrics Pipeline

```
Security Exporters
       │
       ▼
prometheus.scrape  ──▶  prometheus.relabel  ──▶  prometheus.remote_write
                          (add cluster,             (Thanos Receiver /
                           env, tool labels)         Mimir endpoint)
```

### Logs Pipeline

```
Security Tool Log Files / Syslog
       │
       ▼
loki.source.file  ──▶  loki.process  ──▶  loki.write
                         (parse JSON,        (Grafana Loki)
                          extract severity,
                          add tool label)
```

---

## Security Tool Integrations

| Tool | Data Type | Collection Method |
|---|---|---|
| CrowdStrike Falcon | Metrics | prometheus.scrape via exporter |
| Splunk | Logs | loki.source.file (HEC forwarder logs) |
| HashiCorp Vault | Metrics + Audit Logs | prometheus.scrape + loki.source.file |
| Falco | Metrics + Events | prometheus.scrape + JSON log parsing |
| Okta | Logs | HTTP pull via loki.source.api |
| Palo Alto NGFW | Syslog | loki.source.syslog receiver |
| WAF (Cloudflare) | Logs | Logpush → loki.write |
| Tenable | Metrics | prometheus.scrape via exporter |

---

## Forwarding Targets

| Target | Protocol | Data Type |
|---|---|---|
| Thanos Receiver | Prometheus remote_write | Metrics |
| Grafana Loki | Loki push API | Logs |
| Grafana Tempo | OTLP gRPC | Traces |
| Alertmanager | Direct (via Prometheus) | Alerts |

All forwarding connections use TLS with client certificates managed by cert-manager.

---

## Service Discovery

Alloy uses Kubernetes service discovery to automatically detect new exporter pods as they are deployed. Pods must carry the label `prometheus.io/scrape: "true"` and optionally `prometheus.io/port` to be picked up automatically.

This means adding a new cybersecurity exporter to the cluster automatically brings it into the Alloy collection pipeline without manual configuration changes.

---

> **Alloy UI:** When running in Kubernetes, Alloy exposes a local debug UI on port `12345` for inspecting pipeline component health and live data flow. Access it via `kubectl port-forward` — never expose this port externally.
