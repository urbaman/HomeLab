# Monitoring Cert Manager

Deploy the following PodMonitor, after having added the cert-manager namespace to the kube-prometheus-stack deployment

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: cert-manager
  namespace: cert-manager
  labels:
    app: cert-manager
    app.kubernetes.io/name: cert-manager
    app.kubernetes.io/instance: cert-manager
    app.kubernetes.io/component: controller
    release: kube-prometheus-stack
spec:
  jobLabel: app.kubernetes.io/name
  selector:
    matchLabels:
      app: cert-manager
      app.kubernetes.io/name: cert-manager
      app.kubernetes.io/instance: cert-manager
      app.kubernetes.io/component: controller
  podMetricsEndpoints:
    - port: tcp-prometheus-servicemonitor
      honorLabels: true
```

Search and add the cert-manager grafana dashboard

```bash
kubectl create configmap grafana-dashboard-cert-manager --from-file=grafana-cm.json
kubectl label configmap grafana-dashboard-cert-manager grafana_dashboard="1"
```
