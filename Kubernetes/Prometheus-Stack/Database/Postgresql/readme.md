# Add the grafana Dashboard (see installation for metrics exposure)

```bash
kubectl create configmap grafana-dashboard-postgresql -n monitoring --from-file=grafana-postgresql.json
kubectl label configmap grafana-dashboard-postgresql -n monitoring grafana_dashboard="1"
```
