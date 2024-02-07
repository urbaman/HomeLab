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
helm upgrade -i -f prom.yaml kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring
```

Now create Endpoints, Service and Service Monitor (see example yaml) and apply it with kubectl.

Finally, add a grafana dashboard for etcd on grafana, set the datasource, get the json, create a configmap from it and label it.

```bash
kubectl create configmap grafana-dashboard-etcd-cluster -n monitoring --from-file=etcd-cluster.json
kubectl label configmap grafana-dashboard-etcd-cluster -n monitoring grafana_dashboard="1"
kubectl create configmap grafana-dashboard-etcd-cluster-overview -n monitoring --from-file=etcd-cluster-overview.json
kubectl label configmap grafana-dashboard-etcd-cluster-overview -n monitoring grafana_dashboard="1"
```
