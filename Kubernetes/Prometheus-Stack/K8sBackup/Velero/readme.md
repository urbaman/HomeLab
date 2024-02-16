# Grafana Dashboard for Velero

Deploy the dashboard

```bash
kubectl create configmap grafana-dashboard-velero -n monitoring --from-file=grafana-velero.json
kubectl label configmap grafana-dashboard-velero -n monitoring grafana_dashboard="1"
```
