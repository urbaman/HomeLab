# Monitoring Memcached

Remember to activate the servicemonitor in the helm values, and add the namespace to the prometheus operator, then add the dashboards

```bash
kubectl create configmap grafana-dashboard-memcached-pods -n monitoring --from-file=memcached_pods.json
kubectl label configmap grafana-dashboard-memcached-pods -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached -n monitoring --from-file=Memcached.json
kubectl label configmap grafana-dashboard-memcached -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached2 -n monitoring --from-file=Memcached2.json
kubectl label configmap grafana-dashboard-memcached2 -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-memcached3 -n monitoring --from-file=Memcached3.json
kubectl label configmap grafana-dashboard-memcached3 -n monitoring grafana_dashboard="1"
```
