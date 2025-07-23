# Monitoring

## Add the grafana Dashboard (see installation for metrics exposure)

```bash
kubectl create configmap grafana-dashboard-postgresql -n monitoring --from-file=grafana-postgresql.json
kubectl label configmap grafana-dashboard-postgresql -n monitoring grafana_dashboard="1"
```

## Add the grafana dashboard and the prometheusRules for cloudnative pg

```bash
kubectl create configmap grafana-dashboard-cloudnativepg -n monitoring --from-file=grafana-cloudnativepg.json
kubectl label configmap grafana-dashboard-cloudnativepg -n monitoring grafana_dashboard="1"
kubectl apply -f prometheusrules-cloudnativepg.yaml
```
