# Traefik monitoring

Remember to set the metrics port expose setting to true when installing Traefik, and set metrics and servicemonitor to true, with the proper additional labels.

To add a grafana dashboard, first install it in grafana, then download the json from grafana itself, and then create a configmap from it in the monitoring namespace

```bash
kubectl create configmap grafana-dashboard-traefik -n monitoring --from-file=grafana-traefik.json
kubectl label configmap grafana-dashboard-traefik -n monitoring grafana_dashboard="1"
```
