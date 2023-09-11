# Uptime Kuma monitoring

## Create the Prometheus Service Monitor

Create a secret with the basic auth base64 username and password, then the service monitor

```bash
kubectl apply -f sm-ukuma.yaml
```

Then, import and define the grafana dashboard.

```bash
kubectl create configmap grafana-dashboard-uptime-kuma --from-file=grafana-uk.json
kubectl label configmap grafana-dashboard-uptime-kuma grafana_dashboard="1"
```