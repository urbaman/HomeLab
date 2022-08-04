# Monitoring with Prometheus-stack (Prometheus and Grafana)

Start checking the bind address of kube-proxy, kube-scheduler, kube-control-monitor

```bash
kubectl edit cm kube-proxy -n kube-system
## Change from
    metricsBindAddress: 127.0.0.1:10249 ### <--- Too secure
## Change to
    metricsBindAddress: 0.0.0.0:10249
```

And reload kube-proxy deployment

```bash
kubectl delete pod -n kube-ysstem kubepod1 kubepod2 ...
```

On every control-panel, change bind adreess to 0.0.0.0:

```bash
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
sudo vi /etc/kubernetes/manifests/kube-control-manager.yaml
```

Then kill each scheduler and each control manager pods in the kube-system namespace

```bash
kubectl delete pod <pod-name> -n kube-system
```

Get the helm value file and add all your namespaces. Add others and upgrade the deploy if needed.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm show values prometheus-community/kube-prometheus-stack > prom-stack.yaml
```

```bash
  namespaces:
    releaseNamespace: true
    additional:
      - calico-apiserver
      - calico-system
      - cloudflare-operator
      - default
      - kube-node-lease
      - kube-public
      - metallb-system
      - tigera-operator
      - traefik
```

Under the grafana header, add some plugins:

```bash
    plugins:
      - grafana-piechart-panel
      - grafana-clock-panel
```

And chenge the default dashboard timezone to "browser":

```bash
  defaultDashboardsTimezone: browser
```

They will then be managed through the grafana config map:

```bash
kubectl edit cm -n monitoring kube-prometheus-stack-grafana
```

Finally, set the ```defaultDashboardsTimezone: browser``` to get the right timezione in the dashboards.

```bash
helm install --namespace monitoring --create-namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack --values prom-stack.yaml
```

## Monitoring other services/apps on other namespaces

Take a look at the helm values of the prometheus-stack implementation:

```bash
kubectl get kube-prometheus-stack -n monitoring prometheus-kube-prometheus-prometheus -o yaml
```

Near the end you'll find something like:

```bash
  serviceMonitorSelector:
    matchLabels:
      release: kube-prometheus-stack
```

In here, ```release: kube-prometheus-stack``` is the label to add to the Service Monitors you'll need to create for the services to be monitored.

### Example: Traefik (see more below for Traefik implementation)

Remember to set the metrics port expose setting to true when installing Traefik, then create the following Service Monitor:

```bash
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

### Example2: External Etcd Cluster

Copy the etcd certs to a tmp folder:

```bash
mkdir /tmp/etcd-monitor
cp /etc/kubernetes/pki/apiserver-etcd* /tmp/etcd-monitor
cp /etc/kubernetes/pki/etcd/* /tmp/etcd-monitor
```

And create a secret:

```bash
kubectl create secret generic etcd-client-cert -n monitoring --from-file=/tmp/etcd-monitor/ca.crt --from-file=/tmp/etcd-monitor/apiserver-etcd-client.crt --from-file=/tmp/etcd-monitor/apiserver-etcd-client.key
```

Now, disable the internal etcd cluster scraping, and add the secret:

```bash
helm get values -n monitoring prometheus -a -o yaml > prom.yaml
```

Set kubeEtcd enabled: false

```bash
kubeEtcd:
  enabled: false
```

Set secrets (find it under prometheusSpec)

```bash
  prometheusSpec:
    secrets: ["etcd-client-cert"]
```

And upgrade the helm implementation

```bash
helm upgrade -f prom.yaml prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

Now create Endpoints, Service and Service Monitor:

```bash
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

Finally, add a grafana dashboard for etcd on grafana, set trhe datasource, get the json, create a configmap from it and label it.

```bash
kubectl create configmap grafana-dashboard-etcd-cluster --from-file=/path_to/etcd-cluster.json
kubectl label configmap grafana-dashboard-etcd-cluster grafana_dashboard="1"
```

## Exposing Grafana:

```bash
kubectl patch svc "$APP_INSTANCE_NAME-grafana" \
  --namespace "$NAMESPACE" \
  -p '{"spec": {"type": "LoadBalancer"}}'
```
