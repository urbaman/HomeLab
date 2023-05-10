# Cert Manager (to manage SSL certs with Traefik in HA mode)

## Install Cert-manager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
```

## Promheteus monitoring

Deploy the following PodMonitor, after havign added the cert-manager namespace to the kube-prometheus-stack deployment

```
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: cert-manager
  namespace: cert-manager
  labels:
    app: cert-manager
    app.kubernetes.io/name: cert-manager
    app.kubernetes.io/instance: cert-manager
    app.kubernetes.io/component: "controller"
    release: kube-prometheus-stack
spec:
  jobLabel: app.kubernetes.io/name
  selector:
    matchLabels:
      app: cert-manager
      app.kubernetes.io/name: cert-manager
      app.kubernetes.io/instance: cert-manager
      app.kubernetes.io/component: "controller"
  podMetricsEndpoints:
    - port: http-metrics
      honorLabels: true
```
Search and add the cert-manager grafana dashboard

```
kubectl create configmap grafana-dashboard-cert-manager --from-file=grafana-cm.json
kubectl label configmap grafana-dashboard-cert-manager grafana_dashboard="1"
```
