# Proxmox, Prometheus, Grafana

After installing the prometheus exporter on the nodes, define endpoints, service and service monitor:

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: pve-cluster
  labels:
    app: pve
  namespace: monitoring
subsets:
- addresses:
  - ip: 10.0.10.11
    nodeName: pvenode1
  - ip: 10.0.10.12
    nodeName: pvenode2
  - ip: 10.0.10.13
    nodeName: pvenode3
  ports:
  - name: pve
    port: 9221
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: pve-cluster
  labels:
    app: pve
  namespace: monitoring
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: pve
    port: 9221
    targetPort: 9221
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: pve-cluster
  namespace: monitoring
  labels:
    app: pve
    release: kube-prometheus-stack
spec:
  endpoints:
  - interval: 30s
    port: pve
    scheme: http
    path: "/pve"
  jobLabel: pve-cluster
  selector:
    matchLabels:
      app: pve
```

Then, add the dashboard to grafana and create the configmap.
