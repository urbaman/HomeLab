# Couchdb monitoring

Deploy the couchdb exporter

```bash
helm upgrade -i -n couchdb prometheus-couchdb-exporter prometheus-community/prometheus-couchdb-exporter --set couchdb.uri=http://couchdb-couchdb.couchdb.svc:5984 --set couchdb.username=admin --set couchdb.password=tjTXrAQSFythqta9DZ2U --set rbac.pspEnabled=false
```bash

Deploy a servicemonitor

```bash
kubectl appy -f prom-couchdb.yaml
```

## Add the grafana dashboard

```bash
kubectl create configmap grafana-dashboard-mysql --from-file=grafana-mysql.json
kubectl label configmap grafana-dashboard-mysql grafana_dashboard="1"
```
