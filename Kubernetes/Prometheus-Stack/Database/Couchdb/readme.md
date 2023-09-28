# Couchdb monitoring

[See installation](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Coucheb) for metrics enablement

Deploy a servicemonitor

```bash
kubectl apply -f prom-couchdb.yaml
```

## Add the grafana dashboard (not yet found)

```bash
kubectl create configmap grafana-dashboard-couchdb --from-file=grafana-couchdb.json
kubectl label configmap grafana-dashboard-couchdb grafana_dashboard="1"
```
