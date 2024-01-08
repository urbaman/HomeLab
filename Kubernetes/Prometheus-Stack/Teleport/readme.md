# Teleport monitoring

Remember to set podMonitors to True

## Create the Grafana Dashboard

```bash
kubectl create configmap grafana-dashboard-teleport-cluster --from-file=grafana-teleport-cluster.json
kubectl label configmap grafana-dashboard-teleport-cluster grafana_dashboard="1"
```
