# Mongodb monitoring

Remember to activate the servicemonitor in the helm values, and add the namespace to the prometheus operator, then add the dashboards

```bash
kubectl create configmap grafana-dashboard-mongodb -n monitoring --from-file=grafana-mongodb.json
kubectl label configmap grafana-dashboard-mongodb -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-mongodb2 -n monitoring --from-file=grafana-mongodb2.json
kubectl label configmap grafana-dashboard-mongodb2 -n monitoring grafana_dashboard="1"
```
