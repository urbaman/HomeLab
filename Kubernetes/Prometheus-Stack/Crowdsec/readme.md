# Crowdsec monitoring

Remember to deploy Crowdsec with metrics = true and serviceMonitor = true, then add a label `release: prometheus-kube-stack` to the service monitors.

Create the dashboards (now in v5):

```bash
kubectl create configmap grafana-dashboard-crowdsec-dpm -n monitoring --from-file=crowdsec-details-per-machine.json
kubectl label configmap grafana-dashboard-crowdsec-dpm -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-overview -n monitoring --from-file=crowdsec-overview.json
kubectl label configmap grafana-dashboard-crowdsec-overview -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-insight -n monitoring --from-file=crowdsec-insight.json
kubectl label configmap grafana-dashboard-crowdsec-insight -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-crowdsec-lapi -n monitoring --from-file=crowdsec-lapi.json
kubectl label configmap grafana-dashboard-crowdsec-lapi -n monitoring grafana_dashboard="1"
```
