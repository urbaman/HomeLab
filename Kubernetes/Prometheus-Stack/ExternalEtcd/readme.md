# External Etcd Cluster

Copy the etcd certs to a tmp folder:

```bash
sudo mkdir /tmp/etcd-monitor
sudo cp /etc/kubernetes/pki/apiserver-etcd* /tmp/etcd-monitor
sudo cp /etc/kubernetes/pki/etcd/* /tmp/etcd-monitor
sudo chown ubuntu:ubuntu /tmp/etcd-monitor/*
```

And create a secret:

```bash
kubectl create secret generic etcd-client-cert -n monitoring --from-file=/tmp/etcd-monitor/ca.crt --from-file=/tmp/etcd-monitor/apiserver-etcd-client.crt --from-file=/tmp/etcd-monitor/apiserver-etcd-client.key
```

Now, disable the internal etcd cluster scraping, and add the secret:

```bash
helm get values -n monitoring kube-prometheus-stack -a -o yaml > prom.yaml
```

Set kubeEtcd enabled: false

```yaml
kubeEtcd:
  enabled: false
```

Set secrets (find it under prometheusSpec)

```yaml
  prometheusSpec:
    secrets: ["etcd-client-cert"]
```

And upgrade the helm implementation

```bash
helm upgrade -f prom.yaml kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

Now create Endpoints, Service and Service Monitor:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: etcd-cluster
  labels:
    app: etcd
  namespace: monitoring
subsets:
- addresses:
  - ip: 10.0.50.64
    nodeName: etcd
  ports:
  - name: metrics
    port: 2379
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: etcd-cluster
  labels:
    app: etcd
  namespace: monitoring
spec:
  type: ClusterIP
  clusterIP: None
  ports:
  - name: metrics
    port: 2379
    targetPort: 2379
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: etcd-cluster
  namespace: monitoring
  labels:
    app: etcd
    release: kube-prometheus-stack
spec:
  endpoints:
  - interval: 30s
    port: metrics
    scheme: https
    tlsConfig:
      caFile: /etc/prometheus/secrets/etcd-client-cert/ca.crt
      certFile: /etc/prometheus/secrets/etcd-client-cert/apiserver-etcd-client.crt
      keyFile: /etc/prometheus/secrets/etcd-client-cert/apiserver-etcd-client.key
        #serverName: etcdclstr
  jobLabel: etcd-cluster
  selector:
    matchLabels:
      app: etcd
```

And apply it with kubectl.

Finally, add a grafana dashboard for etcd on grafana, set the datasource, get the json, create a configmap from it and label it.

```bash
kubectl create configmap grafana-dashboard-etcd-cluster --from-file=/path_to/etcd-cluster.json
kubectl label configmap grafana-dashboard-etcd-cluster grafana_dashboard="1"
```