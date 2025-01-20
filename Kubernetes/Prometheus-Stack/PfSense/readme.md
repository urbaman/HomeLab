# PfSense with node-exporter

Install the node exporter in PfSense, then apply the endpoint, service and servicemonitor, finally create the dashboard(s).

```bash
kubectl apply -f sm-pfsense.yaml
kubectl create configmap grafana-dashboard-pfsense-node-exporter -n monitoring --from-file=grafana-pfsense.json
kubectl label configmap grafana-dashboard-pfsense-node-exporter -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-pfsense-node-exporter2 -n monitoring --from-file=grafana-pfsense2.json
kubectl label configmap grafana-dashboard-pfsense-node-exporter2 -n monitoring grafana_dashboard="1"
```
