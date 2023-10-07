# Ceph monitoring

## Enable the ceph prometheus module

```bash
ceph mgr module enable prometheus
```

## Install the endpoints, service, service monitor and grafana dashboards

Add the vonage-status-panel and grafana-piechart-panel plugins to the grafana config map, then add dashboards [from here](https://github.com/ceph/ceph/tree/main/monitoring/ceph-mixin/dashboards_out)

```bash
kubectl apply -f prom-ceph.yaml
kubectl create configmap grafana-dashboard-ceph-cluster --from-file=ceph-cluster.json
kubectl label configmap grafana-dashboard-ceph-cluster grafana_dashboard="1"
kubectl create configmap grafana-dashboard-cephfs-overview --from-file=cephfs-overview.json
kubectl label configmap grafana-dashboard-cephfs-overview grafana_dashboard="1"
kubectl create configmap grafana-dashboard-osd-device-details --from-file=osd-device-details.json
kubectl label configmap grafana-dashboard-osd-device-details grafana_dashboard="1"
kubectl create configmap grafana-dashboard-osds-overview --from-file=osds-overview.json
kubectl label configmap grafana-dashboard-osds-overview grafana_dashboard="1"
kubectl create configmap grafana-dashboard-pool-detail --from-file=pool-detail.json
kubectl label configmap grafana-dashboard-pool-detail grafana_dashboard="1"
kubectl create configmap grafana-dashboard-pools-overview --from-file=pools-overview.json
kubectl label configmap grafana-dashboard-pools-overview grafana_dashboard="1"
```
