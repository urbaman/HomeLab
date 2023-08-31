# Traefik monitoring

Remember to set the metrics port expose setting to true when installing Traefik, then create the following Service Monitor:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: traefik-sm
  labels:
    traefik: http
    release: kube-prometheus-stack
spec:
  jobLabel: traefik
  selector:
    matchExpressions:
    - {key: app.kubernetes.io/name, operator: Exists}
  namespaceSelector:
    matchNames:
    - traefik
  endpoints:
  - port: metrics
    interval: 15s
```

To add a grafana dashboard, first install it in grafana, then download the json from grafana itself, and then create a configmap from it in the monitoring namespace

```bash
kubectl create configmap grafana-dashboard-traefik --from-file=/path_to/traefik_dashboard.json
kubectl label configmap grafana-dashboard-traefik grafana_dashboard="1"
```
