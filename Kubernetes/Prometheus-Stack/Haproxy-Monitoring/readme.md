# Haproxy monitoring with prometheus and grafana

## Haproxy prometheus exporter

Be sure to enable the native prometheus exporter in Haproxy (see https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Cluster/03-High-Availability)

## Apply Endpoint, Servce and ServiceMonitor

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: haproxy1
  labels:
    app: haproxy1
  namespace: monitoring
subsets:
- addresses:
  - ip: 10.0.50.61
    nodeName: haproxy1
  ports:
  - name: haproxy1
    port: 8404
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: haproxy1
  labels:
    app: haproxy1
  namespace: monitoring
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: haproxy1
    port: 8404
    targetPort: 8404
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: haproxy1
  namespace: monitoring
  labels:
    app: haproxy1
    release: kube-prometheus-stack
spec:
  endpoints:
  - interval: 30s
    port: haproxy1
    scheme: http
  jobLabel: haproxy1
  selector:
    matchLabels:
      app: haproxy1
```

Then, add Grafana dashboard

```bash
kubectl create configmap grafana-dashboard-haproxy --from-file=grafana-haproxy.json
kubectl label configmap grafana-dashboard-haproxy grafana_dashboard="1"
```
