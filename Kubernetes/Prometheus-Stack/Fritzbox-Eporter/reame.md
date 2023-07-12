# Fritzbox exporter for Prometheus Monitoring

## Apply the prom-fritzbox.yaml file after having customized the environment variables

```bash
kubectl apply -f prom-fritzbox.yaml
```

## Create the Grafana dashboard

```bash
kubectl create configmap grafana-dashboard-fritzbox --from-file=grafana-fritzbox.json
kubectl label configmap grafana-dashboard-fritzbox grafana_dashboard="1"
```
