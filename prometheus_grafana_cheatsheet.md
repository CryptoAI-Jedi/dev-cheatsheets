# prometheus_grafana_cheatsheet

# PROMETHEUS — CORE CONCEPTS

---

Metric Types:
counter     → Always increases (e.g. total requests)
gauge       → Goes up/down (e.g. memory usage)
histogram   → Bucketed observations (e.g. request duration)
summary     → Quantiles over sliding window

Labels         → Key-value pairs on metrics (e.g. {job="node", instance="host:9100"})
Scrape         → Prometheus pulling metrics from /metrics endpoint
Target         → Endpoint Prometheus scrapes
Exporter       → Agent exposing metrics (node_exporter, cadvisor, etc.)
Alert          → Rule that fires when PromQL condition is true
Alertmanager   → Routes alerts to email/Slack/PagerDuty

---

## PROMETHEUS — CONFIG (prometheus.yml)

global:
scrape_interval: 15s
evaluation_interval: 15s

scrape_configs:

- job_name: "node"
static_configs:
    - targets: ["localhost:9100"]
- job_name: "docker"
static_configs:
    - targets: ["localhost:8080"]

rule_files:

- "alert_rules.yml"

alerting:
alertmanagers:
- static_configs:
- targets: ["localhost:9093"]

---

## PROMETHEUS — SERVICE MANAGEMENT

systemctl start prometheus
systemctl status prometheus
systemctl restart prometheus
curl localhost:9090/-/healthy        → Health check
curl localhost:9090/-/reload         → Reload config (POST)
curl -X POST localhost:9090/-/reload → Reload config

---

## PROMQL — QUERYING

# Basic

node_cpu_seconds_total                         → Raw metric
up                                             → 1=target up, 0=down
up{job="node"}                                 → Filter by label
node_memory_MemAvailable_bytes                 → Available RAM in bytes

# Functions

rate(http_requests_total[5m])                  → Per-second rate over 5min
irate(http_requests_total[5m])                 → Instant rate (spiky data)
increase(http_requests_total[1h])              → Total increase over 1hr
sum(rate(http_requests_total[5m])) by (job)    → Aggregate by label
avg(node_load1) by (instance)                  → Average per instance
max_over_time(node_load1[1h])                  → Max value in last hour
histogram_quantile(0.95, rate(http_duration_bucket[5m]))  → p95 latency

# Filtering

node_filesystem_avail_bytes{mountpoint="/"}    → Specific label value
node_filesystem_avail_bytes{job!="node"}       → Exclude label
node_cpu_seconds_total{mode=~"user|system"}    → Regex match
node_cpu_seconds_total{mode!~"idle|iowait"}    → Regex exclude

# Math

node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100  → RAM % free
(node_filesystem_size_bytes - node_filesystem_free_bytes)
/ node_filesystem_size_bytes * 100                               → Disk % used

---

## ALERT RULES (alert_rules.yml)

groups:

- name: node_alerts
rules:
    - alert: HighCPU
    expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
    for: 2m
    labels:
    severity: warning
    annotations:
    summary: "High CPU on {{ $labels.instance }}"
    - alert: LowDiskSpace
    expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes * 100 < 15
    for: 5m
    labels:
    severity: critical
    annotations:
    summary: "Low disk on {{ $labels.instance }}: {{ $value | printf \"%.0f\" }}% free"
    - alert: NodeDown
    expr: up == 0
    for: 1m
    labels:
    severity: critical
    annotations:
    summary: "Node {{ $labels.instance }} is down"

---

## NODE EXPORTER

# Install & run

./node_exporter &
curl localhost:9100/metrics              → Verify metrics exposed

# Key metrics

node_cpu_seconds_total                  → CPU time by mode
node_memory_MemAvailable_bytes          → Free RAM
node_disk_io_time_seconds_total         → Disk I/O
node_network_receive_bytes_total        → Network in
node_network_transmit_bytes_total       → Network out
node_filesystem_avail_bytes             → Disk free space
node_load1 / node_load5 / node_load15   → Load averages

---

## GRAFANA — SETUP

systemctl start grafana-server
systemctl status grafana-server
Default URL: [http://localhost:3000](http://localhost:3000/)
Default login: admin / admin (change on first login)

---

## GRAFANA — DATA SOURCES

UI: Configuration → Data Sources → Add data source → Prometheus
URL: [http://localhost:9090](http://localhost:9090/)
Click "Save & Test" → should show green "Data source is working"

---

## GRAFANA — DASHBOARDS

Import community dashboard:
Dashboards → Import → Enter ID → Load
Node Exporter Full: ID 1860
Docker: ID 193
Kubernetes: ID 315

Create panel:

- New Dashboard → Add panel → Select data source → Enter PromQL → Apply

Panel types:
Time series   → CPU, memory, network over time
Gauge         → Single value with thresholds
Stat          → Bold single number
Table         → Multi-column output
Bar chart     → Comparisons across labels

---

## GRAFANA — USEFUL SETTINGS

Thresholds:     Panel → Field → Thresholds (green/yellow/red)
Alerts:         Panel → Alert tab → Create alert rule
Variables:      Dashboard Settings → Variables → Add (e.g. $instance)
Use in query: node_load1{instance="$instance"}
Auto-refresh:   Top right → set interval (e.g. 10s, 30s)
Time range:     Top right → Last 1h / 6h / 24h

---

## DOCKER COMPOSE (Prometheus + Grafana + Node Exporter)

version: "3"
services:
prometheus:
image: prom/prometheus
ports: ["9090:9090"]
volumes:
- ./prometheus.yml:/etc/prometheus/prometheus.yml

grafana:
image: grafana/grafana
ports: ["3000:3000"]
volumes:
- grafana-data:/var/lib/grafana

node-exporter:
image: prom/node-exporter
ports: ["9100:9100"]

volumes:
grafana-data: