# Grafana Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Installation & Access](#installation--access)
3. [Data Sources](#data-sources)
4. [Dashboards & Panels](#dashboards--panels)
5. [Queries & Transformations](#queries--transformations)
6. [Templating & Variables](#templating--variables)
7. [Alerting](#alerting)
8. [Annotations](#annotations)
9. [Plugins & Visualizations](#plugins--visualizations)
10. [User Management & Permissions](#user-management--permissions)
11. [Configuration](#configuration)
12. [Homelab Use Cases](#homelab-use-cases)
13. [Shortcuts & Tips](#shortcuts--tips)

---

## Basics

```text
Grafana → Visualization & observability platform
Key features:
- Connect to multiple data sources (Prometheus, Loki, InfluxDB, SQL, Elasticsearch)
- Build dashboards with interactive panels
- Alerting & notifications
- Annotations for events/logs
- Dashboard templating for dynamic metrics
- Plugins for extended visualizations
- Explore tab for ad-hoc queries
```

---

## Installation & Access

```bash
# Run Grafana via Docker
docker run -d --name=grafana \
  -p 3000:3000 \
  -e "GF_SECURITY_ADMIN_USER=admin" \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
  grafana/grafana

# Default login
User: admin
Password: admin

# Access web UI
http://localhost:3000
```

* Alternative: Install via package manager (apt/yum) or Helm chart in Kubernetes

---

## Data Sources

```text
# Common data sources:
- Prometheus → Time-series metrics
- Loki → Logs
- InfluxDB → Time-series metrics
- MySQL/PostgreSQL → Relational data
- Elasticsearch → Logs & search
- Graphite → Historical metrics

# Add data source in UI:
Configuration → Data Sources → Add data source → URL & credentials

# Prometheus example:
URL: http://prometheus:9090
```

---

## Dashboards & Panels

```text
# Dashboard → collection of panels
- Can be shared, exported/imported as JSON
- Supports folders for organization

# Panel types
- Graph → Time series visualization
- Stat → Single value
- Gauge → Value indicator
- Table → Tabular data
- Heatmap → Density visualization
- Logs → Log visualization (with Loki)
- Text → Markdown/HTML for notes

# Panel actions
- Edit → Configure query, visualization, thresholds, units
- Duplicate → Copy panel
- Repeat → Repeat panel based on variable
- Fullscreen → Expand for detailed view
```

---

## Queries & Transformations

```text
# Prometheus queries in Grafana

# Instant value
up

# Rate of metric over 5 min
rate(node_cpu_seconds_total[5m])

# Aggregate sum across instances
sum(rate(node_cpu_seconds_total[5m])) by (instance)

# Disk usage %
(node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Top 5 CPU consumers
topk(5, rate(node_cpu_seconds_total[5m]))

# Filter by labels
rate(node_cpu_seconds_total{mode="user"}[5m])

# Transformations
- Add field from calculation (e.g., compute usage percentage)
- Merge fields from multiple queries
- Filter by regex
- Rename columns/labels
```

---

## Templating & Variables

```text
# Variables → dynamic dashboards
- Example: $instance, $job, $disk

# Query variable → populate options via data source query
- e.g., label_values(node_cpu_seconds_total, instance)

# Custom variable → static list
- e.g., server1, server2

# Interval variable → time intervals (1m, 5m, 1h)

# Usage in query
rate(node_cpu_seconds_total{instance="$instance"}[5m])

# Repeat panel per variable
Repeat panel for each $instance
```

---

## Alerting

```text
# Alert rules defined per panel
- Conditions: metric > or < threshold
- Evaluate every X seconds/minutes
- Notifications: Email, Slack, Teams, Webhook

# Example:
CPU usage > 80% for 5 minutes → trigger alert

# Integration:
- Prometheus Alertmanager
- Grafana Notification channels

# Alert properties:
- Name
- Condition
- Evaluation interval
- For → duration to avoid flapping
- Labels → severity, environment
- Annotations → summary, description
```

---

## Annotations

```text
# Mark events on dashboards
- Example: Server reboot, deployment, backup
- Types: Manual or automatic via queries
- Can display text and links
- Improves correlation between events and metrics
```

---

## Plugins & Visualizations

```text
# Extend Grafana capabilities
- Panel plugins → additional charts, heatmaps, maps
- Data source plugins → connect to new sources
- App plugins → dashboards, alerts, features

# Install via CLI or UI
grafana-cli plugins install <plugin-id>

# Common plugins:
- Worldmap Panel
- Pie Chart
- Clock Panel
- Status Panel
```

---

## User Management & Permissions

```text
# User roles
- Admin → full access
- Editor → create/edit dashboards
- Viewer → read-only dashboards

# Teams
- Group users and assign dashboard/folder permissions

# Permissions
- Folder-level → edit/view dashboards
- Dashboard-level → granular access
```

---

## Configuration

```text
# General settings:
- Organization → Teams & users
- Preferences → Timezone, theme, home dashboard
- Plugins → Add visualization or data source plugins
- Snapshots → Share dashboard state (read-only)
```

* Export dashboard JSON:

```text
Dashboard → Settings → JSON Model → Export
```

* Import dashboard JSON:

```text
Create → Import → Upload JSON or paste
```

---

## Homelab Use Cases

```text
# Node exporter → CPU, memory, disk metrics
# SNMP exporter → Routers, switches, UPS
# Blackbox exporter → Ping critical services
# Docker/Podman metrics → Containers & resource usage
# Alerts → CPU, memory, disk, network thresholds
# Dynamic dashboards → Use variables for multiple hosts/services
# Annotations → Server restarts, backup events, deployments
```

---

## Shortcuts & Tips

| Shortcut / Feature        | Description                                        |
| ------------------------- | -------------------------------------------------- |
| `Shift + Click` on legend | Show/hide series                                   |
| `$variable`               | Template variable in queries                       |
| Panel → Thresholds        | Highlight warning/critical levels                  |
| Panel → Transformations   | Modify or join metrics                             |
| Dashboard → JSON          | Export/import dashboards                           |
| Explore tab               | Ad-hoc queries & debugging                         |
| Repeat panel              | Duplicate panel by variable                        |
| Ctrl + S                  | Save dashboard                                     |
| Ctrl + Z / Ctrl + Y       | Undo/redo                                          |
| Fullscreen                | Expand panel/dashboard                             |
| Refresh                   | Auto-refresh interval (5s, 10s, 1m, etc.)          |
| Link panels               | Drill-down to another dashboard or URL             |
| Row / Panel collapse      | Improve dashboard readability                      |
| Annotations → Query       | Automatically show events from database/Prometheus |

