# Monitoring metallb

After implementing the prometheus-stack (see below), get the prometheus-operator.yaml file from the github project, add the label ```release: kube-prometheus-stack``` to both the Pod Monitors and apply.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: metallb-controller
  labels:
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      component: controller
  namespaceSelector:
    matchNames:
      - metallb-system
  podMetricsEndpoints:
    - port: monitoring
      path: /metrics
---
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: metallb-speaker
  labels:
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      component: speaker
  namespaceSelector:
    matchNames:
      - metallb-system
  podMetricsEndpoints:
    - port: monitoring
      path: /metrics
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: prometheus-k8s
  namespace: metallb-system
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: prometheus-k8s
  namespace: metallb-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: prometheus-k8s
subjects:
  - kind: ServiceAccount
    name: prometheus-k8s
    namespace: monitoring
```

Install the grafana dashboard, download the json from grafana, create the configmap in the monitoring namespace and label it.

```bash
kubectl create configmap grafana-dashboard-metallb -n monitoring --from-file=/path_to/metallb_dashboard.json
kubectl label configmap grafana-dashboard-metallb -n monitoring grafana_dashboard="1"
```
