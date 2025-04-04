# Kured monitoring

If you deploy with kubectl, also deploy the servicemonitor.

```bash
kubectl apply -f servicemonitor-kured.yaml
```

If you deploy with helm, set metrics and servicemonitor to true and set the release label.

Import the kured Grafana dashboard, fix the datasource id to match "prometheus" and the namespace variable to match "kube-system", then copy the json content to a grafana-kured.json file.

```bash
kubectl create configmap grafana-dashboard-kured -n monitoring --from-file=grafana-kured.json
kubectl label configmap grafana-dashboard-kured -n monitoring grafana_dashboard="1"
```
