# Ceph monitoring

## Enable the ceph prometheus module

```bash
ceph mgr module enable prometheus
```

## Install the endpoints, service, service monitor and grafana dashboards

Add the vonage-status-panel and grafana-piechart-panel plugins to the grafana config map, then add dashboards [from here](https://github.com/ceph/ceph/tree/main/monitoring/ceph-mixin/dashboards_out)

```bash
kubectl apply -f prom-ceph.yaml
kubectl create configmap grafana-dashboard-name --from-file=filename.json
kubectl label configmap grafana-dashboard-name grafana_dashboard="1"
```
