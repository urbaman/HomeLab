# Redis monitoring

See redis deployment for metrics and servicemonitor enablement, then create the dashboard

```bash
kubectl create configmap grafana-dashboard-redis --from-file=grafana-redis.json
kubectl label configmap grafana-dashboard-redis grafana_dashboard="1"
```
