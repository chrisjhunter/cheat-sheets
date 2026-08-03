# Prometheus / PromQL Cheat Sheet

## Basic selectors
```promql
up                                    # instant vector: 1 if target is up, 0 if down
http_requests_total                     # all time series for this metric
http_requests_total{job="api"}            # filter by label
http_requests_total{job="api", status=~"5.."}   # regex label match
http_requests_total{status!="200"}                # negative match
http_requests_total{job=~"api|web"}                 # OR via regex
```

## Range vectors & rate
```promql
http_requests_total[5m]                        # raw samples over last 5 minutes
rate(http_requests_total[5m])                    # per-second average rate (for counters)
irate(http_requests_total[5m])                     # instant rate, last two points (spiky metrics)
increase(http_requests_total[1h])                    # total increase over 1 hour
```

## Aggregation
```promql
sum(rate(http_requests_total[5m]))                          # total request rate across all instances
sum by (job) (rate(http_requests_total[5m]))                  # grouped by label
sum(rate(http_requests_total{status=~"5.."}[5m])) by (job)      # error rate per job
avg(node_memory_MemAvailable_bytes) by (instance)
max(node_load1) by (instance)
count(up == 1)                                                     # number of healthy targets
topk(5, sum by (job) (rate(http_requests_total[5m])))                # top 5 busiest jobs
```

## Common SRE queries
```promql
# Error rate percentage
100 * sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# CPU usage percentage (node_exporter)
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

# Disk usage percentage
100 * (1 - node_filesystem_avail_bytes / node_filesystem_size_bytes)

# 95th percentile request latency (histogram)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Targets currently down
up == 0
```

## Math & comparisons
```promql
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024      # bytes -> GB
rate(http_requests_total[5m]) > 100                         # filter series above a threshold
delta(cpu_temp_celsius[10m])                                   # difference over time (gauges)
predict_linear(node_filesystem_avail_bytes[1h], 4 * 3600)        # predict value 4h from now (e.g. disk-full ETA)
```

## Alerting rule example
```yaml
groups:
  - name: example
    rules:
      - alert: HighErrorRate
        expr: |
          100 * sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 5
        for: 10m
        labels:
          severity: page
        annotations:
          summary: "Error rate above 5% for 10m"
```

## promtool CLI
```bash
promtool check config prometheus.yml         # validate main config
promtool check rules alerts.yml                # validate alerting/recording rules
promtool query instant http://localhost:9090 'up'
promtool query range http://localhost:9090 'rate(http_requests_total[5m])' --start=... --end=...
```

## Useful one-liners
```bash
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | select(.health!="up")'  # unhealthy targets
curl -s "http://localhost:9090/api/v1/query?query=up" | jq .            # query via HTTP API
curl -s http://localhost:9090/-/healthy                                    # Prometheus's own health check
curl -s http://localhost:9090/api/v1/label/__name__/values | jq .             # list all metric names
```
