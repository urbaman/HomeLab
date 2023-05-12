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
kubectl delete pod -n kube-system kubepod1 kubepod2 ...
```

On every control-panel, change bind address to 0.0.0.0:

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

```yaml
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

Disable the internal etcd scraping:

```yaml
kubeEtcd:
  enabled: false
```

And define Alertmanager notifications by email:

```yaml
    route:
      group_by: ['namespace']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 12h
      receiver: 'null'
      routes:
      - receiver: 'null'
        matchers:
          - alertname =~ "InfoInhibitor|Watchdog"
      - receiver: 'mail'
    receivers:
    - name: 'null'
    - name: 'mail'
      email_configs:
      - to: 'tomail@domain.com'
        from: 'frommail@domain.com'
        smarthost: smtp.domain.com:587
        auth_username: 'tomail@domain.com'
        auth_identity: 'tomail@domain.com'
        auth_password: 'password'
        send_resolved: true
```

Under the grafana header, add some plugins:

```yaml
    plugins:
      - grafana-piechart-panel
      - grafana-clock-panel
```

They will then be managed through the grafana config map:

```bash
kubectl edit cm -n monitoring kube-prometheus-stack-grafana
```

Finally, set the ```defaultDashboardsTimezone: browser``` to get the right timezone in the dashboards.

```bash
helm install --namespace monitoring --create-namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack --values prom-stack.yaml
```

## Monitoring other services/apps on other namespaces

Take a look at the helm values of the prometheus-stack implementation:

```bash
kubectl get kube-prometheus-stack -n monitoring prometheus-kube-prometheus-prometheus -o yaml
```

Near the end you'll find something like:

```yaml
  serviceMonitorSelector:
    matchLabels:
      release: kube-prometheus-stack
```

In here, ```release: kube-prometheus-stack``` is the label to add to the Service Monitors you'll need to create for the services to be monitored.

### Example: Traefik (see more below for Traefik implementation)

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
helm upgrade -f prom.yaml prometheus prometheus-community/kube-prometheus-stack -n monitoring
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

Finally, add a grafana dashboard for etcd on grafana, set trhe datasource, get the json, create a configmap from it and label it.

```bash
kubectl create configmap grafana-dashboard-etcd-cluster --from-file=/path_to/etcd-cluster.json
kubectl label configmap grafana-dashboard-etcd-cluster grafana_dashboard="1"
```

## Exposing services with Traefik and cert-manager

### Grafana (admin prom-operator)

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: grafana-urbaman
  namespace: monitoring
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - grafana.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: grafana-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-grafana-basic-auth
  namespace: monitoring
spec:
  basicAuth:
    secret: monitoring-grafana-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-grafana-https-redirect
  namespace: monitoring
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-grafana-security
  namespace: monitoring
spec:
  headers:
    frameDeny: true
    sslRedirect: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 31536000
---
apiVersion: traefik.io/v1alpha1
kind: ServersTransport
metadata:
  name: monitoring-grafana-transport
  namespace: monitoring
spec:
  serverName: grafana
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: monitoring-grafana-tlsoptions
  namespace: monitoring
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_AES_128_GCM_SHA256
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_FALLBACK_SCSV
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: false
---
apiVersion: v1
kind: Secret
metadata:
  name: monitoring-grafana-basic-auth
  namespace: monitoring
data:
  users: |
    YWRtaW46JGFwcjEkSVBqUU96bmYkWkhDSWp2UmdTNFd4bXBUZnVVakhMMQoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-grafana-websecure
  namespace: monitoring
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`grafana.urbaman.it`)
      services:
      - name: kube-prometheus-stack-grafana
        port: 80
        serversTransport: monitoring-grafana-transport
      middlewares:
        - name: monitoring-grafana-basic-auth
        - name: monitoring-grafana-security
  tls:
    secretName: grafana-urbaman
    options:
      name: monitoring-grafana-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-grafana-web
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`grafana.urbaman.it`)
      services:
      - name: kube-prometheus-stack-grafana
        port: 80
      middlewares:
        - name: monitoring-grafana-https-redirect
```

### Prometheus (admin prom-operator)

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: prometheus-urbaman
  namespace: monitoring
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - prometheus.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: prometheus-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-prometheus-basic-auth
  namespace: monitoring
spec:
  basicAuth:
    secret: monitoring-prometheus-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-prometheus-https-redirect
  namespace: monitoring
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-prometheus-security
  namespace: monitoring
spec:
  headers:
    frameDeny: true
    sslRedirect: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 31536000
---
apiVersion: traefik.io/v1alpha1
kind: ServersTransport
metadata:
  name: monitoring-prometheus-transport
  namespace: monitoring
spec:
  serverName: prometheus
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: monitoring-prometheus-tlsoptions
  namespace: monitoring
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_AES_128_GCM_SHA256
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_FALLBACK_SCSV
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: false
---
apiVersion: v1
kind: Secret
metadata:
  name: monitoring-prometheus-basic-auth
  namespace: monitoring
data:
  users: |
    YWRtaW46JGFwcjEkSVBqUU96bmYkWkhDSWp2UmdTNFd4bXBUZnVVakhMMQoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-prometheus-websecure
  namespace: monitoring
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`prometheus.urbaman.it`)
      services:
      - name: kube-prometheus-stack-prometheus
        port: 9090
        serversTransport: monitoring-prometheus-transport
      middlewares:
        - name: monitoring-prometheus-basic-auth
        - name: monitoring-prometheus-security
  tls:
    secretName: prometheus-urbaman
    options:
      name: monitoring-prometheus-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-prometheus-web
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`prometheus.urbaman.it`)
      services:
      - name: kube-prometheus-stack-prometheus
        port: 9090
      middlewares:
        - name: monitoring-prometheus-https-redirect
```

### Alertmanager (admin prom-operator)

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: alertmanager-urbaman
  namespace: monitoring
spec:
  # Certificate will be valid for these domain names
  dnsNames:
  - alertmanager.urbaman.it
  # Reference our issuer
  # As it's a ClusterIssuer, it can be in a different namespace
  issuerRef:
    kind: ClusterIssuer
    name: cert-manager-acme-issuer
  # Secret that will be created with our certificate and private keys
  secretName: alertmanager-urbaman
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-alertmanager-basic-auth
  namespace: monitoring
spec:
  basicAuth:
    secret: monitoring-alertmanager-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-alertmanager-https-redirect
  namespace: monitoring
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: monitoring-alertmanager-security
  namespace: monitoring
spec:
  headers:
    frameDeny: true
    sslRedirect: true
    browserXssFilter: true
    contentTypeNosniff: true
    stsIncludeSubdomains: true
    stsPreload: true
    stsSeconds: 31536000
---
apiVersion: traefik.io/v1alpha1
kind: ServersTransport
metadata:
  name: monitoring-alertmanager-transport
  namespace: monitoring
spec:
  serverName: alertmanager
  insecureSkipVerify: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: TLSOption
metadata:
  name: monitoring-alertmanager-tlsoptions
  namespace: monitoring
spec:
  minVersion: VersionTLS12
  cipherSuites:
    - TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
    - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305
    - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305
    - TLS_AES_256_GCM_SHA384
    - TLS_AES_128_GCM_SHA256
    - TLS_CHACHA20_POLY1305_SHA256
    - TLS_FALLBACK_SCSV
  curvePreferences:
    - CurveP521
    - CurveP384
  sniStrict: false
---
apiVersion: v1
kind: Secret
metadata:
  name: monitoring-alertmanager-basic-auth
  namespace: monitoring
data:
  users: |
    YWRtaW46JGFwcjEkSVBqUU96bmYkWkhDSWp2UmdTNFd4bXBUZnVVakhMMQoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-alertmanager-websecure
  namespace: monitoring
spec:
  entryPoints:
    - websecure
  routes:
    - kind: Rule
      match: Host(`alertmanager.urbaman.it`)
      services:
      - name: kube-prometheus-stack-alertmanager
        port: 9090
        serversTransport: monitoring-alertmanager-transport
      middlewares:
        - name: monitoring-alertmanager-basic-auth
        - name: monitoring-alertmanager-security
  tls:
    secretName: alertmanager-urbaman
    options:
      name: monitoring-alertmanager-tlsoptions
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: monitoring-alertmanager-web
  namespace: monitoring
spec:
  entryPoints:
    - web
  routes:
    - kind: Rule
      match: Host(`alertmanager.urbaman.it`)
      services:
      - name: kube-prometheus-stack-alertmanager
        port: 9090
      middlewares:
        - name: monitoring-alertmanager-https-redirect
```
