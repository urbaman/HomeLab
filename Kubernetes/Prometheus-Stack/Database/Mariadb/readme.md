# Mariadb Monitoring

Add the grafana Dashboard (see installation for metrics exposure)

```bash
kubectl create configmap grafana-dashboard-mysql -n monitoring --from-file=grafana-mysql.json
kubectl label configmap grafana-dashboard-mysql -n monitoring grafana_dashboard="1"
```
