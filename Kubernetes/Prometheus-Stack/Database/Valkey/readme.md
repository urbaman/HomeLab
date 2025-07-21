# Valkey monitoring

See valkey deployment for metrics and servicemonitor enablement, then create the dashboard

```bash
kubectl create configmap grafana-dashboard-valkey -n monitoring --from-file=grafana-valkey.json
kubectl label configmap grafana-dashboard-valkey -n monitoring grafana_dashboard="1"
```
