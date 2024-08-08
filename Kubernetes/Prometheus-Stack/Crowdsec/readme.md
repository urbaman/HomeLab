# Crowdsec monitoring

Remember to deploy Crowdsec with metrics = true and serviceMonitor = true, then add a label `release: prometheus-kube-stack` to the service monitors.

Create the dashboards (now in v5):

```bash
kubectl create configmap grafana-dashboard-crowdsec-dpm --from-file=crowdsec-details-per-machine.json
kubectl label configmap grafana-dashboard-crowdsec-dpm grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-overview --from-file=crowdsec-overview.json
kubectl label configmap grafana-dashboard-crowdsec-overview grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-insight --from-file=crowdsec-insight.json
kubectl label configmap grafana-dashboard-crowdsec-insight grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-lapi --from-file=crowdsec-lapi.json
kubectl label configmap grafana-dashboard-crowdsec-lapi grafana_dashboard="1"
```
