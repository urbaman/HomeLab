# Monitoring Memcached

```bash
kubectl create configmap grafana-dashboard-memcached-pods --from-file=memcached_pods.json
kubectl label configmap grafana-dashboard-memcached-pods grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached --from-file=Memcached.json
kubectl label configmap grafana-dashboard-memcached grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached2 --from-file=Memcached2.json
kubectl label configmap grafana-dashboard-memcached2 grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached3 --from-file=Memcached3.json
kubectl label configmap grafana-dashboard-memcached3 grafana_dashboard="1"
```