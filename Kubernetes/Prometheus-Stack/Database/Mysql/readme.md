# Add the grafana Dashboard (see installation for metrics exposure)

```bash
kubectl create configmap grafana-dashboard-mysql --from-file=grafana-mysql.json
kubectl label configmap grafana-dashboard-mysql grafana_dashboard="1"
```
