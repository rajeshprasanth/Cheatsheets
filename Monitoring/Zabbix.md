# Zabbix Cheat Sheet

## Table of Contents

1. [Basics](#basics)
2. [Zabbix CLI](#zabbix-cli)
3. [Zabbix API](#zabbix-api)
4. [Hosts, Templates & Groups](#hosts-templates--groups)
5. [Items](#items)
6. [Triggers](#triggers)
7. [Graphs, Screens & Dashboards](#graphs-screens--dashboards)
8. [Macros](#macros)
9. [Actions & Alerting](#actions--alerting)
10. [Maintenance](#maintenance)
11. [Reporting & Monitoring](#reporting--monitoring)
12. [Troubleshooting](#troubleshooting)
13. [Homelab Use Cases](#homelab-use-cases)

---

## Basics

```text
Zabbix → Enterprise monitoring solution
- Monitor: servers, VMs, network devices, applications
- Components: Server, Agent, Proxy, Frontend
- Protocols: SNMP, IPMI, JMX, HTTP, SSH, custom scripts
- Features:
  - Alerts & notifications
  - Templates & host groups
  - Time-series data collection
  - Graphs, dashboards, screens
  - API for automation
```

---

## Zabbix CLI

```bash
# Server version
zabbix_server -V

# Agent test key
zabbix_agentd -t <key> -p <port> <host>

# Reload agent
sudo systemctl reload zabbix-agent

# Get item value from agent
zabbix_get -s <host> -k <key>

# Send custom value to Zabbix server
zabbix_sender -z <server> -s "<hostname>" -k "<key>" -o "<value>"

# Check active agent configuration
zabbix_agentd -p
```

---

## Zabbix API

```bash
# API URL: http://<zabbix-server>/api_jsonrpc.php

# Authentication
POST JSON:
{
  "jsonrpc": "2.0",
  "method": "user.login",
  "params": {"user": "Admin", "password": "zabbix"},
  "id": 1,
  "auth": null
}

# Create host
method: host.create
params: {
  "host": "server1",
  "interfaces": [{"type":1,"main":1,"useip":1,"ip":"10.0.0.1","dns":"","port":"10050"}],
  "groups": [{"groupid":"2"}],
  "templates":[{"templateid":"10001"}]
}

# List hosts
method: host.get
params: { "output": ["hostid","host","name"] }

# Update host
method: host.update
params: { "hostid":"10105", "name":"new-name" }

# Delete host
method: host.delete
params: ["10105"]
```

---

## Hosts, Templates & Groups

```text
# Hosts → devices being monitored
- IP/DNS, agent type, port
- Assign templates
- Group for logical organization

# Templates → reusable items/triggers/graphs
- Examples: Linux OS, Windows OS, SNMP Devices
- Link multiple templates to host

# Host Groups → organize hosts
- Examples: Servers, Network Devices, Lab
- Assign permissions per group
```

---

## Items

```text
# Items → metrics collected from hosts
- Key → metric identifier
- Type: Zabbix agent, SNMP, IPMI, JMX, External, Simple check, HTTP
- Update interval → polling frequency
- History & trends retention

# Example:
- Key: system.cpu.load[percpu,avg1]
- Type: Zabbix agent
- Interval: 60s

# Item status:
- Enabled / Disabled
```

---

## Triggers

```text
# Triggers → conditions for alerting
- Expression syntax: {host:key.last()} > 80
- Severity: Not classified, Information, Warning, Average, High, Disaster
- Dependencies → prevent duplicate alerts
- Recovery expression → optional

# Example:
- Trigger: High CPU Load
  Expression: {Server1:system.cpu.load[percpu,avg1].last()} > 80
```

---

## Graphs, Screens & Dashboards

```text
# Graphs
- Visualize single or multiple items
- Types: Line, Stacked, Bar, Pie

# Screens
- Combine multiple graphs/items
- Useful for status overview

# Dashboards
- Multiple panels
- Variables for dynamic metrics
- Shareable & exportable (JSON)
```

---

## Macros

```text
# Macros → placeholders for dynamic values
- User macros: {$VAR_NAME} → used in triggers, items
- Built-in macros: {HOST.NAME}, {HOST.IP}
- Alert message macros: {TRIGGER.NAME}, {ITEM.VALUE}, {EVENT.TIME}
```

---

## Actions & Alerting

```text
# Actions → define alerts & operations
- Trigger → condition
- Operation → send message, run remote command
- Recovery operation → optional
- Escalation → multiple steps

# Notifications channels:
- Email, Slack, Telegram, Webhook
```

---

## Maintenance

```text
# Put hosts into maintenance mode
- Web UI → Configuration → Hosts → Maintenance
- Schedule maintenance
- Prevent alerts during maintenance
```

---

## Reporting & Monitoring

```text
# Built-in reports
- Availability report
- SLA report
- Audit logs

# Monitoring dashboards
- Current problems
- Latest data per host/item
- Event history
```

---

## Troubleshooting

```bash
# Agent logs
tail -f /var/log/zabbix/zabbix_agentd.log

# Server logs
tail -f /var/log/zabbix/zabbix_server.log

# Debug agent
zabbix_agentd -t <key> -p <port> -f

# Test SNMP connectivity
snmpwalk -v2c -c public <host>
```

---

## Homelab Use Cases

```text
# Monitor servers & VMs → CPU, memory, disk, network
# Monitor network devices via SNMP → Switch ports, router interfaces
# Monitor UPS → battery, voltage, runtime remaining
# Monitor Docker/Podman containers → uptime, resource usage
# Dynamic dashboards → multiple hosts using templates and variables
# Alerts → email/Slack/Telegram for high load or downtime
# Integrate with Grafana → better visualization of metrics
```

