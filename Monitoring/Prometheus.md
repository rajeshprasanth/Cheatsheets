# Prometheus Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Installation & Targets](#installation--targets)
3. [Scraping Metrics](#scraping-metrics)
4. [PromQL Queries](#promql-queries)
5. [Recording Rules](#recording-rules)
6. [Alerting Rules](#alerting-rules)
7. [Configuration](#configuration)
8. [Advanced Features](#advanced-features)
9. [Prometheus Web UI](#prometheus-web-ui)
10. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
Prometheus → Monitoring & alerting system
Key features:
- Pull-based metrics collection
- Multi-dimensional data model (labels)
- Powerful query language (PromQL)
- Alertmanager integration
- Exporters for Node, SNMP, MySQL, PostgreSQL, Redis, Docker, etc.
- Federation & multi-datacenter monitoring
```

---

## Installation & Targets

```bash
# Run Prometheus via Docker
docker run -d --name prometheus \
  -p 9090:9090 \
  -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus

# Check scrape targets
http://localhost:9090/targets

# Add exporters:
# node_exporter → server metrics
# snmp_exporter → network devices
# blackbox_exporter → ping/http checks
```

---

## Scraping Metrics

```yaml
# prometheus.yml example
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'snmp_exporter'
    static_configs:
      - targets: ['192.168.1.1:9116']

  - job_name: 'docker'
    metrics_path: /metrics
    static_configs:
      - targets: ['docker-host:9323']
```

* `job_name` → logical group
* `metrics_path` → endpoint path
* `relabel_configs` → rename labels dynamically

---

## PromQL Queries

### Instant Queries

```text
# Target health
up

# CPU usage per core
node_cpu_seconds_total

# Memory available
node_memory_MemAvailable_bytes

# Disk usage
node_filesystem_avail_bytes / node_filesystem_size_bytes * 100
```

### Range Queries

```text
# CPU usage over last 5 min
rate(node_cpu_seconds_total[5m])

# Disk I/O per second
rate(node_disk_reads_completed_total[1m])
rate(node_disk_writes_completed_total[1m])
```

### Aggregation

```text
# Sum CPU usage across all cores
sum(rate(node_cpu_seconds_total[5m])) by (instance)

# Average memory available by instance
avg(node_memory_MemAvailable_bytes) by (instance)

# Top 5 highest CPU consumers
topk(5, rate(node_cpu_seconds_total[5m]))
```

### Filtering with Labels

```text
# CPU usage for mode="user"
rate(node_cpu_seconds_total{mode="user"}[5m])

# Disk usage for /home
node_filesystem_avail_bytes{mountpoint="/home"} / node_filesystem_size_bytes{mountpoint="/home"} * 100
```

---

## Recording Rules

```yaml
groups:
- name: node.rules
  interval: 1m
  rules:
  - record: job:node_cpu:rate5m
    expr: rate(node_cpu_seconds_total[5m])
  - record: job:node_memory:available_bytes
    expr: node_memory_MemAvailable_bytes
```

* Recorded metrics reduce query load
* Use `record` to define precomputed metrics

---

## Alerting Rules

```yaml
groups:
- name: node_alerts
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} has been down >5min."

  - alert: HighCPU
    expr: sum(rate(node_cpu_seconds_total[5m])) by (instance) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU > 80% for more than 2 min."
```

* Alerts integrate with **Alertmanager** for notifications

---

## Configuration

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

* `scrape_interval` → how often to scrape metrics
* `evaluation_interval` → how often to evaluate rules

---

## Advanced Features

```text
# Federation
- Prometheus server can scrape another Prometheus server
- Useful for multi-datacenter aggregation

# Blackbox Exporter
- Check HTTP endpoints, ICMP ping, TCP ports

# Recording rules
- Precompute expensive queries for dashboards

# Remote Write
- Send metrics to long-term storage like Thanos, Cortex

# Grafana integration
- Connect Prometheus as a datasource
- Build dashboards and alerts
```

---

## Prometheus Web UI

```text
# Access UI
http://localhost:9090

# Features:
- Execute PromQL queries
- Graph metrics over time
- Explore targets & status
- View recording and alerting rules
```

---

## Homelab Use Cases

```text
# Node exporter
- Monitor CPU, memory, disk usage for homelab servers

# SNMP exporter
- Monitor routers, switches, UPS

# Blackbox exporter
- Ping critical services like web servers, database nodes

# Alerts
- Notify if server down, high CPU, low disk space
- Integrate with Grafana for dashboards

# Example alert: Disk space < 10%
expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes * 100) < 10
```

---

## Useful Command Summary

| Command                                      | Description                   |
| -------------------------------------------- | ----------------------------- |
| `docker run -d -p 9090:9090 prom/prometheus` | Start Prometheus container    |
| `up`                                         | Check target health           |
| `rate(<metric>[5m])`                         | Compute rate over 5 minutes   |
| `sum(rate(...)) by(instance)`                | Aggregate metrics by instance |
| `record`                                     | Define recording rule         |
| `alert`                                      | Define alerting rule          |
| `targets`                                    | List scrape targets           |
| `/metrics`                                   | Exporter metrics endpoint     |
| `topk(5, <metric>)`                          | Show top 5 values             |
| `avg(<metric>) by (<label>)`                 | Average metric by label       |
