# Authentik monitoring

Set metrics and servicemonitor to true in the helm chart values, then add the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-authentik -n monitoring --from-file=grafana-authentik.json
kubectl label configmap grafana-dashboard-authentik -n monitoring grafana_dashboard="1"
```
