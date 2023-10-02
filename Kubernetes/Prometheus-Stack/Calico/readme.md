# Apply patches to enable the metrics

```bash
kubectl patch felixconfiguration default --type merge --patch '{"spec":{"prometheusMetricsEnabled": true}}'
kubectl patch installation default --type=merge -p '{"spec": {"typhaMetricsPort":9093}}'
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: felix-metrics-svc
  namespace: calico-system
  labels:
    k8s-app: calico-node
spec:
  clusterIP: None
  selector:
    k8s-app: calico-node
  ports:
  - name: metrics-port
    port: 9091
    targetPort: 9091
EOF
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: typha-metrics-svc
  namespace: calico-system
  labels:
    k8s-app: calico-typha
spec:
  clusterIP: None
  selector:
    k8s-app: calico-typha
  ports:
  - name: metrics-port
    port: 9093
    targetPort: 9093
EOF
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: calico-controllers-sm
  namespace: calico-system
  labels:
    release: kube-prometheus-stack
spec:
  jobLabel: calico-controllers
  namespaceSelector:
    matchNames:
    - calico-system
  selector:
    matchLabels:
      k8s-app: calico-kube-controllers
  endpoints:
  - port: metrics-port
    interval: 15s
    path: '/metrics'
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: felix-metrics-svc-sm
  namespace: calico-system
  labels:
    release: kube-prometheus-stack
    k8s-app: calico-node
spec:
  jobLabel: felix-metrics
  namespaceSelector:
    matchNames:
    - calico-system
  selector:
    matchLabels:
      k8s-app: calico-node
  endpoints:
  - port: metrics-port
    interval: 15s
    path: '/metrics'
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: typha-metrics-svc-sm
  namespace: calico-system
  labels:
    release: kube-prometheus-stack
    k8s-app: calico-typha
spec:
  jobLabel: typha-metrics
  selector:
    matchLabels:
      k8s-app: calico-typha
  endpoints:
  - port: metrics-port
    interval: 15s
    path: '/metrics'
EOF
kubectl create configmap grafana-dashboard-felix --from-file=grafana-felix.json
kubectl label configmap grafana-dashboard-felix grafana_dashboard="1"
kubectl create configmap grafana-dashboard-typha --from-file=grafana-typha.json
kubectl label configmap grafana-dashboard-typha grafana_dashboard="1"
```
