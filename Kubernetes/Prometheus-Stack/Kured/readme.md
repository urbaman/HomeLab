# Kured monitoring

```bash
kubectl apply -f prom-kured.yaml
```

Import the kured Grafana dashboard, fix the datasource id to match "prometheus" and the namespace variable to match "kube-system", then copy the json content to a grafana-kured.json file.

```bash
kubectl create configmap grafana-dashboard-kured --from-file=grafana-kured.json
kubectl label configmap grafana-dashboard-kured grafana_dashboard="1"
```
