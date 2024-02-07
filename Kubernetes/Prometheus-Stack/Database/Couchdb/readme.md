# Couchdb monitoring

Deploy the couchdb exporter

```bash
helm upgrade -i -n couchdb prometheus-couchdb-exporter prometheus-community/prometheus-couchdb-exporter --set couchdb.uri=http://couchdb-couchdb.couchdb.svc:5984 --set couchdb.username=admin --set couchdb.password=<PASSWORD> --set rbac.pspEnabled=false
```bash

Deploy a servicemonitor

```bash
kubectl apply -f prom-couchdb.yaml
```

## Add the grafana dashboard (not yet found)

```bash
kubectl create configmap grafana-dashboard-couchdb -n monitoring --from-file=grafana-couchdb.json
kubectl label configmap grafana-dashboard-couchdb -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-couchdb2 -n monitoring --from-file=grafana-couchdb2.json
kubectl label configmap grafana-dashboard-couchdb2 -n monitoring grafana_dashboard="1"
```
