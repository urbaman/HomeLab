# Couchdb monitoring

[See installation](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Coucheb) for metrics enablement

Deploy a servicemonitor

```bash
kubectl appy -f prom-couchdb.yaml
```

## Add the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-mysql --from-file=grafana-mysql.json
kubectl label configmap grafana-dashboard-mysql grafana_dashboard="1"
```
