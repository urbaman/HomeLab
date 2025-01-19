# Teleport monitoring

Remember to set podMonitors to True

## Create the Grafana Dashboard

```bash
kubectl create configmap grafana-dashboard-teleport-cluster -n monitoring --from-file=grafana-teleport-cluster.json
kubectl label configmap grafana-dashboard-teleport-cluster -n monitoring grafana_dashboard="1"
```
