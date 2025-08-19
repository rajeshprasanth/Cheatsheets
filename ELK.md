# **ELK Stack Cheat Sheet**

## **Table of Contents**

1. [Overview](#overview)
2. [Elasticsearch](#elasticsearch)

   * [Core Concepts](#core-concepts)
   * [Basic Commands](#basic-commands)
   * [Advanced Queries](#advanced-queries)
3. [Logstash](#logstash)

   * [Pipeline Structure](#pipeline-structure)
   * [Sample Config](#sample-config)
   * [Filters & Plugins](#filters--plugins)
4. [Kibana](#kibana)

   * [Key Features](#key-features)
   * [KQL Queries](#kql-queries)
   * [Dashboards & Visualizations](#dashboards--visualizations)
5. [Beats](#beats)

   * [Filebeat](#filebeat)
   * [Metricbeat](#metricbeat)
   * [Other Beats](#other-beats)
6. [Monitoring & Maintenance](#monitoring--maintenance)
7. [Troubleshooting](#troubleshooting)
8. [Homelab Use Cases](#homelab-use-cases)
9. [Useful Commands](#useful-commands)

---

## **Overview**

```text
The ELK Stack (Elasticsearch, Logstash, Kibana) is used for:
- Centralized logging
- Metrics collection & analytics
- Security monitoring (SIEM)
- Application & infrastructure observability
- Supports structured & unstructured data
```

---

## **Elasticsearch**

### **Core Concepts**

```text
- Node → single Elasticsearch instance
- Cluster → multiple nodes forming a group
- Index → collection of documents (like a database)
- Document → JSON object stored in an index
- Shard → partition of an index
- Replica → copy of shard for redundancy
```

### **Basic Commands**

```bash
# List indices
curl -X GET "localhost:9200/_cat/indices?v"

# Create an index
curl -X PUT "localhost:9200/logs-2025-08"

# Insert a document
curl -X POST "localhost:9200/logs-2025-08/_doc/1" -H 'Content-Type: application/json' -d '
{
  "timestamp": "2025-08-20T12:00:00",
  "level": "INFO",
  "message": "System started"
}'

# Search documents
curl -X GET "localhost:9200/logs-2025-08/_search" -H 'Content-Type: application/json' -d '
{
  "query": { "match": { "level": "INFO" } }
}'
```

### **Advanced Queries**

```json
# Range query
{
  "query": {
    "range": {
      "timestamp": { "gte": "now-1d/d", "lt": "now/d" }
    }
  }
}

# Aggregation example
{
  "size": 0,
  "aggs": {
    "levels": { "terms": { "field": "level.keyword" } }
  }
}
```

---

## **Logstash**

### **Pipeline Structure**

```text
- Input → Data source (file, beats, syslog)
- Filter → Data transformation (grok, date, mutate)
- Output → Destination (Elasticsearch, stdout, Kafka)
```

### **Sample Config**

```yaml
input {
  file {
    path => "/var/log/syslog"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{WORD:loglevel} %{GREEDYDATA:msg}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "syslog-%{+YYYY.MM.dd}"
  }
  stdout { codec => rubydebug }
}
```

### **Filters & Plugins**

```text
- grok → parse unstructured text
- mutate → rename, remove, replace fields
- date → parse timestamps
- geoip → add location info
- metrics → generate counters or summaries
- multiple outputs → Elasticsearch, file, Kafka
```

---

## **Kibana**

### **Key Features**

```text
- Discover → Search raw logs
- Visualize → Build graphs/charts
- Dashboard → Combine multiple visualizations
- Dev Tools → Run Elasticsearch queries
- Machine Learning → Anomaly detection (X-Pack)
```

### **KQL Queries**

```text
# Basic match
level: "ERROR"

# Combine conditions
level: "ERROR" AND host: "server1"

# Range queries
timestamp >= "2025-08-20T00:00:00" AND timestamp <= "2025-08-20T23:59:59"

# Wildcards
message: "*timeout*"
```

### **Dashboards & Visualizations**

```text
- Types: Line, Bar, Pie, Data Table, Heatmap
- Filters & time selectors
- Variables & dynamic dashboards
- Embed/export dashboards for reports
```

---

## **Beats**

### **Filebeat**

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/*.log

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "filebeat-%{+yyyy.MM.dd}"
```

### **Metricbeat**

```yaml
metricbeat.modules:
  - module: system
    metricsets:
      - cpu
      - memory
      - network
      - disk
    period: 10s
    hosts: ["localhost:9200"]

output.elasticsearch:
  hosts: ["localhost:9200"]
```

### **Other Beats**

* Packetbeat → Network packet capture
* Heartbeat → Service uptime monitoring
* Auditbeat → Security events

---

## **Monitoring & Maintenance**

```text
- Cluster health: green / yellow / red
- Index lifecycle management → delete/archive old logs
- Snapshot & restore → backup Elasticsearch data
- Resource monitoring → CPU, RAM, disk for nodes
```

---

## **Troubleshooting**

```bash
# Check Elasticsearch cluster health
curl -X GET "localhost:9200/_cluster/health?pretty"

# Check node stats
curl -X GET "localhost:9200/_nodes/stats?pretty"

# Test Logstash configuration
logstash -t -f /etc/logstash/conf.d/logstash.conf

# Tail logs
tail -f /var/log/logstash/logstash-plain.log
tail -f /var/log/kibana/kibana.log
```

---

## **Homelab Use Cases**

```text
- Centralized logs for homelab servers (dc01plpmon001, dc01plpbth001, etc.)
- Monitor Docker/Podman container logs
- Track network device logs via syslog
- Alerting with Watcher / custom scripts
- Dashboards combining server metrics, UPS, and network stats
- Correlate events across multiple hosts for troubleshooting
```

---

## **Useful Commands**

```bash
# Elasticsearch
curl -X GET "localhost:9200/_cat/nodes?v"
curl -X GET "localhost:9200/_cat/indices?v"

# Logstash
systemctl start logstash
systemctl enable logstash
logstash -t -f /etc/logstash/conf.d/

# Kibana
systemctl start kibana
systemctl enable kibana
```

