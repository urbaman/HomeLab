# Monitoring Longhorn

Import a suitable grafana dashboard (Longhorn Example is ok), get the json definition and create the json definition file.

```bash
kubectl apply -f sm-longhorn.yaml
kubectl create configmap grafana-dashboard-longhorn --from-file=grafana-longhorn.json
kubectl label configmap grafana-dashboard-longhorn grafana_dashboard="1"
```
