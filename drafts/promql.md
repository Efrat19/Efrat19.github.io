sum(rate(container_cpu_usage_seconds_total{namespace="legacy",pod_name=~"maps(.*)"}[5m])) by (cluster)
