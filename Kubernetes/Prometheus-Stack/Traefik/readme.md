# Traefik monitoring

Remember to set the metrics port expose setting to true when installing Traefik, then create the Service Monitor from the template.

To add a grafana dashboard, first install it in grafana, then download the json from grafana itself, and then create a configmap from it in the monitoring namespace

```bash
kubectl create configmap grafana-dashboard-traefik --from-file=grafana-traefik.json
kubectl label configmap grafana-dashboard-traefik grafana_dashboard="1"
```
