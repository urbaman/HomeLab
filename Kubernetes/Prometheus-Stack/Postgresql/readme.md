# Add the grafana Dashboard (see installation for metrics exposure)

```bash
kubectl create configmap grafana-dashboard-postgresql --from-file=grafana-postgresql.json
kubectl label configmap grafana-dashboard-postgresql grafana_dashboard="1"
```
