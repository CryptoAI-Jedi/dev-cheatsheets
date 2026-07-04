# Prometheus & Grafana Cheatsheet

> Metrics pipeline: exporters expose `/metrics`, Prometheus scrapes and stores, PromQL queries, Grafana visualizes, Alertmanager wakes you up. Pull-based where the server comes to the metrics.

---

## Table of Contents
- [Prometheus — Core Concepts](#prometheus--core-concepts)
- [Prometheus — Config](#prometheus--config)
- [Prometheus — Service Management](#prometheus--service-management)
- [PromQL — Querying](#promql--querying)
- [Alert Rules](#alert-rules)
- [Node Exporter](#node-exporter)
- [Grafana — Setup](#grafana--setup)
- [Grafana — Dashboards](#grafana--dashboards)
- [Docker Compose Stack](#docker-compose-stack)
- [Tips & Gotchas](#tips--gotchas)

---

## PROMETHEUS — CORE CONCEPTS

```text
Metric types:
counter     → Always increases (e.g. total requests) — query with rate()
gauge       → Goes up/down (e.g. memory usage) — query directly
histogram   → Bucketed observations (e.g. request duration)
summary     → Quantiles over sliding window

Labels         → Key-value pairs on metrics ({job="node", instance="host:9100"})
Scrape         → Prometheus pulling metrics from a /metrics endpoint
Target         → Endpoint Prometheus scrapes
Exporter       → Agent exposing metrics (node_exporter, cadvisor, etc.)
Alert          → Rule that fires when a PromQL condition holds
Alertmanager   → Routes alerts to email/Slack/PagerDuty
```

---

## PROMETHEUS — CONFIG

```yaml
# prometheus.yml
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
```

---

## PROMETHEUS — SERVICE MANAGEMENT

```bash
systemctl start prometheus
systemctl status prometheus
systemctl restart prometheus
journalctl -u prometheus -e            # Logs (see systemd sheet)

curl localhost:9090/-/healthy          # Health check
curl -X POST localhost:9090/-/reload   # Reload config WITHOUT restart
# ⚠️ /-/reload requires Prometheus started with --web.enable-lifecycle;
# otherwise use systemctl restart (or SIGHUP: kill -HUP $(pgrep prometheus))

promtool check config prometheus.yml   # Validate config BEFORE reloading
promtool check rules alert_rules.yml   # Validate alert rules
```

---

## PROMQL — QUERYING

### Basics

```text
node_cpu_seconds_total                 → Raw metric
up                                     → 1 = target up, 0 = down
up{job="node"}                         → Filter by label
node_memory_MemAvailable_bytes         → Available RAM in bytes
```

### Functions

```text
rate(http_requests_total[5m])                  → Per-second rate over 5min
irate(http_requests_total[5m])                 → Instant rate (spiky data)
increase(http_requests_total[1h])              → Total increase over 1hr
sum(rate(http_requests_total[5m])) by (job)    → Aggregate by label
avg(node_load1) by (instance)                  → Average per instance
max_over_time(node_load1[1h])                  → Max value in last hour
topk(5, rate(http_requests_total[5m]))         → Top 5 series
histogram_quantile(0.95, rate(http_duration_bucket[5m]))   → p95 latency
```

### Filtering

```text
node_filesystem_avail_bytes{mountpoint="/"}    → Specific label value
node_filesystem_avail_bytes{job!="node"}       → Exclude label
node_cpu_seconds_total{mode=~"user|system"}    → Regex match
node_cpu_seconds_total{mode!~"idle|iowait"}    → Regex exclude
```

### Math

```text
node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100   → RAM % free

(node_filesystem_size_bytes - node_filesystem_free_bytes)
  / node_filesystem_size_bytes * 100                                → Disk % used
```

---

## ALERT RULES

```yaml
# alert_rules.yml
groups:
  - name: node_alerts
    rules:
      - alert: HighCPU
        expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 2m                      # Must hold for 2m before firing
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
```

---

## NODE EXPORTER

```bash
curl localhost:9100/metrics             # Verify metrics exposed
# Run it as a systemd unit, not ./node_exporter & — the systemd sheet's
# service template applies directly (Restart=always, User=node_exporter)
```

### Key metrics

```text
node_cpu_seconds_total                  → CPU time by mode
node_memory_MemAvailable_bytes          → Free RAM
node_disk_io_time_seconds_total         → Disk I/O
node_network_receive_bytes_total        → Network in
node_network_transmit_bytes_total       → Network out
node_filesystem_avail_bytes             → Disk free space
node_load1 / node_load5 / node_load15   → Load averages
```

---

## GRAFANA — SETUP

```bash
systemctl start grafana-server
systemctl status grafana-server
```

```text
Default URL:    http://localhost:3000
Default login:  admin / admin (change on first login)

⚠️ Grafana on a VPS is an internal dashboard — bind/proxy it behind
nginx allow/deny or serve it over the tailnet only. Same containment
pattern as any dashboard (see nginx + Tailscale sheets).
```

### Data source

```text
UI: Configuration → Data Sources → Add data source → Prometheus
URL: http://localhost:9090
"Save & Test" → green "Data source is working"
```

---

## GRAFANA — DASHBOARDS

```text
Import community dashboard:
Dashboards → Import → Enter ID → Load
  Node Exporter Full → ID 1860
  Docker             → ID 193
  Kubernetes         → ID 315

Create panel:
New Dashboard → Add panel → Select data source → Enter PromQL → Apply

Panel types:
Time series   → CPU, memory, network over time
Gauge         → Single value with thresholds
Stat          → Bold single number
Table         → Multi-column output
Bar chart     → Comparisons across labels
```

### Useful settings

```text
Thresholds:    Panel → Field → Thresholds (green/yellow/red)
Alerts:        Panel → Alert tab → Create alert rule
Variables:     Dashboard Settings → Variables → Add (e.g. $instance)
               Use in query: node_load1{instance="$instance"}
Auto-refresh:  Top right → set interval (10s, 30s, ...)
Time range:    Top right → Last 1h / 6h / 24h
```

---

## DOCKER COMPOSE STACK

```yaml
# compose.yaml — Prometheus + Grafana + Node Exporter
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "127.0.0.1:9090:9090"        # Loopback — front with nginx (see Docker sheet)
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    restart: unless-stopped

  grafana:
    image: grafana/grafana
    ports:
      - "127.0.0.1:3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter
    ports:
      - "127.0.0.1:9100:9100"
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
```

---

## TIPS & GOTCHAS

- **`rate()` on a gauge is meaningless** — rate is for counters; gauges are read directly (or with `avg_over_time`). The reverse mistake — graphing a raw counter — produces an ever-climbing line that says nothing; counters always go through `rate()`/`increase()`.
- **Range must cover ≥2 scrape intervals** — `rate(x[15s])` with a 15s scrape interval has one sample and returns nothing. Rule of thumb: range ≥ 4× scrape_interval (`[1m]` minimum at 15s).
- **Label cardinality is the memory killer** — a label with unbounded values (user ID, request path, tx hash) creates a time series per value and eats the server. Labels are for low-cardinality dimensions only.
- **`/-/reload` silently fails without `--web.enable-lifecycle`** — the curl returns an error but it's easy to miss in scripts; `promtool check config` + reload (or restart) is the safe pipeline.
- **`for:` prevents pager fatigue** — an alert without a `for:` duration fires on a single scrape blip. Even `for: 1m` filters most noise.
- **Retention is finite** — default storage retention is ~15 days; capacity-plan or set `--storage.tsdb.retention.time` deliberately before you need last month's data and don't have it.
- **Everything in this stack is an internal service** — Prometheus (9090), node_exporter (9100), and Grafana (3000) all default to unauthenticated HTTP. Loopback/tailnet binding isn't optional hardening; it's the deployment.

---
*Last Updated: 2026-07*
