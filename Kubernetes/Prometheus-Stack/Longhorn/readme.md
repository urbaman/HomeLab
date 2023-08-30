# Monitoring Longhorn

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: longhorn-prometheus-servicemonitor
  namespace: monitoring
  labels:
    name: longhorn-prometheus-servicemonitor
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      app: longhorn-manager
  namespaceSelector:
    matchNames:
    - longhorn-system
  endpoints:
  - port: manager
```

Import a suitable grafana dashboard (Longhorn Example is ok), get the json definition and create the json definition file.

```bash
kubectl create configmap grafana-dashboard-longhorn --from-file=grafana-longhorn.json
kubectl label configmap grafana-dashboard-longhorn grafana_dashboard="1"
```
