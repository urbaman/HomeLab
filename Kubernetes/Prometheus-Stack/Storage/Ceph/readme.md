# Ceph monitoring

## Enable the ceph prometheus module

```bash
ceph mgr module enable prometheus
```

## Install the endpoints, service, service monitor and grafana dashboards

Add the vonage-status-panel and grafana-piechart-panel plugins to the grafana config map, then add dashboards [from here](https://github.com/ceph/ceph/tree/main/monitoring/ceph-mixin/dashboards_out)

```bash
kubectl apply -f prom-ceph.yaml
kubectl create configmap grafana-dashboard-ceph-cluster -n monitoring --from-file=ceph-cluster.json
kubectl label configmap grafana-dashboard-ceph-cluster -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-cephfs-overview -n monitoring --from-file=cephfs-overview.json
kubectl label configmap grafana-dashboard-cephfs-overview -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-osd-device-details -n monitoring --from-file=osd-device-details.json
kubectl label configmap grafana-dashboard-osd-device-details -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-osds-overview -n monitoring --from-file=osds-overview.json
kubectl label configmap grafana-dashboard-osds-overview -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-pool-detail -n monitoring --from-file=pool-detail.json
kubectl label configmap grafana-dashboard-pool-detail -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-pools-overview -n monitoring --from-file=pools-overview.json
kubectl label configmap grafana-dashboard-pools-overview -n monitoring grafana_dashboard="1"
```
