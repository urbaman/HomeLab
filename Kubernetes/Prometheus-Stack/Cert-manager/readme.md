# Monitoring Cert Manager

Deploy the PodMonitor with the prom-certmanager.yaml file, after having added the cert-manager namespace to the kube-prometheus-stack-operator deployment

Search and add the cert-manager grafana dashboard

```bash
kubectl create configmap grafana-dashboard-cert-manager --from-file=grafana-certmanagerm.json
kubectl label configmap grafana-dashboard-cert-manager grafana_dashboard="1"
```
