# Monitoring Cert Manager

Deploy the PodMonitor with the `podmonitor-certmanager.yaml` file, after having added the cert-manager namespace to the kube-prometheus-stack-operator deployment

Search and add the cert-manager grafana dashboard

```bash
kubectl create configmap grafana-dashboard-cert-manager -n monitoring --from-file=grafana-certmanager.json
kubectl label configmap grafana-dashboard-cert-manager -n monitoring grafana_dashboard="1"
```
