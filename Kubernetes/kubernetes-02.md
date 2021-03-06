# Kubernetes: more tools

## Metallb

### Preparation

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

You can achieve this by editing kube-proxy config in current cluster:

```bash
kubectl edit configmap -n kube-system kube-proxy
```

and set:

```bash
mode: "ipvs"
ipvs:
  strictARP: true
```

You can also add this configuration snippet to your kubeadm-config, just append it with --- after the main configuration.

If you are trying to automate this change, these shell snippets may help you:

```bash
# see what changes would be made, returns nonzero returncode if different
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl diff -f - -n kube-system

# actually apply the changes, returns nonzero returncode on errors only
kubectl get configmap kube-proxy -n kube-system -o yaml | \
sed -e "s/strictARP: false/strictARP: true/" | \
kubectl apply -f - -n kube-system
```

### Installation By Manifest

To install MetalLB, apply the manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.3/config/manifests/metallb-native.yaml
```

### Configuring L2 configuration

Layer 2 mode is the simplest to configure: in many cases, you don’t need any protocol-specific configuration, only IP addresses.

Layer 2 mode does not require the IPs to be bound to the network interfaces of your worker nodes. It works by responding to ARP requests on your local network directly, to give the machine’s MAC address to clients.

In order to advertise the IP coming from an IPAddressPool, an L2Advertisement instance must be associated to the IPAddressPool.

For example, the following configuration gives MetalLB control over IPs from 10.0.50.100 to 10.0.50.199 (free ip range on the same network of the k8s cluster hosts, so that che services will be accessible on that network), and configures Layer 2 mode:

```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: metallb-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.50.100-10.0.50.199
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: metallb-l2-advertisement
  namespace: metallb-system
```

Setting no IPAddressPool selector in an L2Advertisement instance is interpreted as that instance being associated to all the IPAddressPools available.

So in case there are specialized IPAddressPools, and only some of them must be advertised via L2, the list of IPAddressPools we want to advertise the IPs from must be declared (alternative, a label selector can be used).

```bash
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: metallb-l2-advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
  - metallb-pool
```

### Monitoring

After implementing the prometheus-stack (see below), get the prometheus-operator.yaml file from the github project, add the label ```release: kube-prometheus-stack``` to both the Pod Monitors and apply.

```bash
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

## Storage (GlusterFS)

### glusterfs-client

Fist you need to install the glusterfs-client package on your nodes. The client is used by the kubernetes scheduler to create the glusterfs volumes.

```bash
sudo apt install glusterfs-client
```

### Discovering GlusterFS in Kubernetes

GlusterFS cluster should be discovered in the Kubernetes cluster. To do that, you need to add an Endpoints object that points to the servers of the GlusterFS cluster.

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1
      - ip: 10.0.50.22
        hostname: truenas2
      - ip: 10.0.50.23
        hostname: truenas3
    ports:
      - port: 1
```

To persist the Glusterfs endpoints, you also need to create a relative service.

```bash
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster 
spec:
  ports:
  - port: 1
```

### Using GlusterFS in Kubernetes

Create the desired path in the glusterfs volume (HDD5T/path/... in my situation)

#### Method 1 — Connecting to GlusterFS directly with Pod manifest

To connect to the GlusterFS volume directly with Pod manifest, use the GlusterfsVolumeSource in the Pod. Here is an example (create the path in the glusterfs volume - HDD5T in my situation):

```bash
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: alpine
      image: alpine:latest
      command:
        - touch
        - /data/test
      volumeMounts:
        - name: glusterfs-volume
          mountPath: /data
  volumes:
    - name: glusterfs-volume
      glusterfs:
        endpoints: glusterfs-cluster
        path: HDD5T/path/...
        readOnly: no
```

#### Method 2 — Connecting using the PersistentVolume resource

To create the PersistentVolume object for the GlusterFS volume, use the following manifest. The storage size does not take any effect. Be sure to name the claim you'll bind it to.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-path
  namespace: project-namespace
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/path/...
    readOnly: false
  claimRef:
    name: glusterfs-path-pvc
    namespace: project-namespace
  persistentVolumeReclaimPolicy: Retain
```

Then create the relative PersistentVolumeClaim (be sure to name the PersistentVolume to Claim):

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: glusterfs-path-pvc
    namespace: project-namespace
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 10Gi
    storageClassName: ""
    volumeName: glusterfs-path
```

And then, use the claim as a volume in the pods:

```bash
apiVersion: v1
kind: Pod
metadata:
    name: mypod
    namespace: project-namespace
spec:
    containers:
      - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: mypod
    volumes:
      - name: mypod
        persistentVolumeClaim:
          claimName: glusterfs-path-pvc
```

## Helm

### From Apt (Debian/Ubuntu)

Members of the Helm community have contributed a Helm package for Apt. This package is generally up to date.

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### From Snap

The Snapcrafters community maintains the Snap version of the Helm package:

```bash
sudo snap install helm --classic
```

## Cloudflare DynDNS and DNS records

To manage cloudflare DNS records we use cloudflare-operator from containeroo. It will check the external public IP of the cluster and create/update selected records (A and CNAME) in all of our cloudflare account.

NOTE: backup any zone/domain/records you'll want to keep, we'll need to manage them through cloudflare-operator.

### Prerequisites

- Install Helm version 3 or later
- Install a supported version of Kubernetes

### Steps

#### Add the containeroo Helm repository (This repository is the only supported source of containeroo charts)

```bash
helm repo add containeroo https://charts.containeroo.ch
```

Update your local Helm chart repository cache:

```bash
helm repo update
```

#### Install CustomResourceDefinitions

cloudflare-operator requires a number of CRD resources, which must be installed manually using kubectl.

```bash
kubectl apply -f https://github.com/containeroo/cloudflare-operator/releases/download/v0.2.3/crds.yaml
```

#### Install cloudflare-operator

To install the cloudflare-operator Helm chart, use the Helm install command as described below (check last version)

```bash
helm install \
  cloudflare-operator containeroo/cloudflare-operator \
  --namespace cloudflare-operator \
  --create-namespace \
  --version v0.2.4
```

### Preparation

Create a secret with your Cloudflare global API Key. The key containing the API key must be named apiKey.

```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: cloudflare-global-api-key
  namespace: cloudflare-operator
stringData:
  apiKey: 1234
EOF
```

Create an Account object:

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: Account
metadata:
  name: account-sample
spec:
  email: mail@example.com
  globalAPIKey:
    secretRef:
      name: cloudflare-global-api-key
      namespace: cloudflare-operator
  managedZones:
    - example.com
EOF
```

Create an IP object for your root domain to automatically update your external IPv4 address:

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: IP
metadata:
  name: dynamic-external-ipv4-address
spec:
  type: dynamic
  interval: 5m
  ipSources:
    - url: https://ifconfig.me/ip
    - url: https://ipecho.net/plain
    - url: https://myip.is/ip/
    - url: https://checkip.amazonaws.com
    - url: https://api.ipify.org
EOF
```

Create a DNSRecord with type A for your root domain:

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: root-domain
  namespace: cloudflare-operator
spec:
  name: example.com
  type: A
  ipRef:
    name: dynamic-external-ipv4-address
  proxied: true
  ttl: 1
  interval: 5m
EOF
```

Annotate all ingresses with your root domain, so cloudflare-operator can create DNSRecords for all hosts specified in all ingresses:

```bash
kubectl annotate ingress \
      --all-namespaces \
      --all \
      "cf.containeroo.ch/content=example.com"
```

### Additional DNSRecord - Internal services

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: vpn
  namespace: cloudflare-operator
spec:
  name: vpn.example.com
  type: A
  ipRef:
    name: dynamic-external-ipv4-address
  proxied: false
  ttl: 120
  interval: 5m
EOF
```

Because you linked the DNSRecord to your IP object (spec.ipRef.name), cloudflare-operator will also update the content of the Cloudflare DNS record for you, if your external IPv4 address changes.

Set the ttl to 120 to shorten waiting time if your external IPv4 address has changed. Set proxied to false because Cloudflare cannot proxy vpn traffic.

### Additional DNSRecord - External Service

Let's say you have an external website blog.example.com hosted on a cloud VPC and your external cloud instance IP address is 178.4.20.69.

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: blog
  namespace: cloudflare-operator
spec:
  name: blob.example.com
  content: 178.4.20.69
  type: A
  proxied: true
  ttl: 1
  interval: 5m
EOF
```

Now your blog will be routed through Cloudflare.

### Additional DNSRecord - CNAME

```bash
kubectl apply -f - << EOF
apiVersion: cf.containeroo.ch/v1beta1
kind: DNSRecord
metadata:
  name: blog
  namespace: cloudflare-operator
spec:
  name: blob.example.com
  content: arecord.example.com
  type: CNAME
  proxied: true
  ttl: 1
  interval: 5m
EOF
```

### Monitoring

TO add Cloudflare Operator to Prometheus/Grafana monitoring, get back here after having implemented the stack, and create a podmonitor and a grafana dashboard

```bash
wget https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Edit the podmonitor yaml file to add the ```release: kube-prometheus-stack``` label

```bash
kubectl apply -f https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Install the dashboard in grafana, download the json and create a configmap from it in the monitoring namespace.

```bash
kubectl create configmap grafana-dashboard-cloudflare-operator -n monitoring --from-file=/tmp/grafana-dashboard-cloudflare-operator.json
```

Add label so Grafana can fetch the dashboard

```bash
kubectl label configmap grafana-dashboard-cloudflare-operator -n monitoring grafana_dashboard="1"
```

## Monitoring with Prometheus-stack

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

They will then be managed through the grafana config map:

```bash
kubectl edit cm -n monitoring kube-prometheus-stack-grafana
```

Finally, set the ```defaultDashboardsTimezone: browser``` to get the right timezone in the dashboards.

```bash
helm install --namespace monitoring --create-namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack --values prom-stack.yaml
```

### Monitoring other services/apps on other namespaces

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

#### Example: Traefik (see more below for Traefik implementation)

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

#### Example2: External Etcd Cluster

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

Finally, add a grafana dashboard for etcd on grafana, set the datasource, get the json, create a configmap from it and label it.

```bash
kubectl create configmap grafana-dashboard-etcd-cluster --from-file=/path_to/etcd-cluster.json
kubectl label configmap grafana-dashboard-etcd-cluster grafana_dashboard="1"
```

### Exposing Grafana

```bash
kubectl patch svc "$APP_INSTANCE_NAME-grafana" \
  --namespace "$NAMESPACE" \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

## Traefik

### Create the glusterfs endpoints in the traefik namespace

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  namespace: traefik
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1
      - ip: 10.0.50.22
        hostname: truenas2
      - ip: 10.0.50.23
        hostname: truenas3
    ports:
      - port: 1
---
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster
  namespace: traefik
spec:
  ports:
  - port: 1
```

### Create the path in the glusterfs volume (from a node in the glusterfs cluster)

```bash
cd /cluster/HDD5T
mkdir k8sstorage
cd k8sstorage
mkdir traefik-ssl
```

#### Give the directory the right ownership

```bash
chown 65532:65532 /cluster/HDD5T/k8sstorage/traefik-ssl
```

### Create the pv and the pvc

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: traefik-ssl-volume
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 128Mi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/traefik-ssl
    readOnly: false
  claimRef:
    name: traefik-ssl-claim
    namespace: traefik
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: traefik-ssl-claim
    namespace: traefik
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 128Mi
    storageClassName: ""
    volumeName: traefik-ssl-volume
```

### Get the helm chart

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
```

### Modify the values to see the persistent volume claim

```bash
helm show values traefik/traefik > traefik.yaml
vi traefik.yaml
```

Go to the persistence section, turn it to true, uncomment the existingClaim line and set it to the claim above.

```bash
persistence:
  accessMode: ReadWriteMany
  annotations: {}
  enabled: true
  existingClaim: traefik-ssl-claim
  name: ssl-certs
  path: /ssl-certs
  size: 128Mi
```

Go to the ports section, and set to true both the metrics and the traefik (dashboard) expose settings. Also add a redirect from web to websecure and the default tls resolver:

```bash
ports:
  metrics:
    expose: true
    exposedPort: 9100
    port: 9100
    protocol: TCP
  traefik:
    expose: true
    exposedPort: 9000
    port: 9000
    protocol: TCP
  web:
    expose: true
    exposedPort: 80
    port: 8000
    protocol: TCP
    redirectTo: websecure
  websecure:
    expose: true
    exposedPort: 443
    port: 8443
    protocol: TCP
    tls:
      certResolver: "cloudflare"
      domains: []
      enabled: true
      options: ""
```

Enable the dashboard ingress route:

```bash
ingressRoute:
  dashboard:
    annotations: {}
    enabled: true
    labels: {}
```

Enable the ingressClass, set it to default:

```bash
ingressClass:
  enabled: true
  fallbackApiVersion: ""
  isDefaultClass: true
```

Finally go to the deployment section, and add the following initContainer to properly manage certificates permissions in case of errors (it will probably not run and break traefik's start process):

```bash
deployment:
  additionalContainers: []
  additionalVolumes: []
  annotations: {}
  enabled: true
  imagePullSecrets: []
  initContainers:
    - name: volume-permissions
      image: busybox:1.31.1
      command: ["sh", "-c", "chmod -Rv 600 /ssl-certs/*"]
      volumeMounts:
        - name: ssl-certs
          mountPath: /ssl-certs
```

### Create SSL certs for your domains

Create a secret with your own Cloudlfare email and API Key:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-dnschallenge-credentials
  namespace: traefik
type: Opaque
stringData:
  email: yourmail@example.com
  apiKey: yourglobalapikey
```

Also in the traefik vales yaml file, add the following additionalArguments and the env settings:

```bash
additionalArguments:
# DNS Challenge
# Cloudflare
  - --certificatesresolvers.cloudflare.acme.dnschallenge.provider=cloudflare
  - --certificatesresolvers.cloudflare.acme.email=yourmail@example.com
  - --certificatesresolvers.cloudflare.acme.dnschallenge.resolvers=1.1.1.1
  - --certificatesresolvers.cloudflare.acme.storage=/ssl-certs/acme-cloudflare.json

```

```bash
env:
# DNS Challenge Credentials
# Cloudflare:
- name: CF_API_EMAIL
  valueFrom:
    secretKeyRef:
      key: email
      name: cloudflare-dnschallenge-credentials
- name: CF_API_KEY
  valueFrom:
    secretKeyRef:
      key: apiKey
      name: cloudflare-dnschallenge-credentials
```

### Install

```bash
helm install traefik traefik/traefik --values traefik.yaml -n traefik
```

### Check the dashboard

Go to a machine with a browser and kubectl installed and properly configured, open the terminal

```bash
kubectl -n traefik port-forward traefik-667459fcbf-h7cqf 9000:9000
```

Now point the browser to <http://localhost:9000/dashboard/> (remember the final slash)

### Proxy an internal service (https redirect, basic auth)

Generate a username/password base64 hash to use in the basicauth secret:

```bash
htpasswd -nb username password | base64
dXNlcm5hbWU6JGFwcjEkLmtMR05oTzAkcnZVZTZBY0k3R2dyRHlFbkI0d2J2LwoK
```

Customize and apply the following yaml file (this is for the k8s dashboard previously exposed, can point to any exposed service):

```bash
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: k8dashboard-basic-auth
  namespace: kubernetes-dashboard
spec:
  basicAuth:
    secret: k8dashboard-basic-auth
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: k8dashboard-https-redirect
  namespace: kubernetes-dashboard
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: v1
kind: Secret
metadata:
  name: k8dashboard-basic-auth
  namespace: kubernetes-dashboard
data:
  users: |
    dXNlcm5hbWU6JGFwcjEkLmtMR05oTzAkcnZVZTZBY0k3R2dyRHlFbkI0d2J2LwoK
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: k8dashboard-websecure
  namespace: kubernetes-dashboard
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`k8dashboard.example.com`)
    kind: Rule
    services:
    - name: kubernetes-dashboard
      port: 80
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: k8dashboard-web
  namespace: kubernetes-dashboard
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`k8dashboard.example.com`)
    kind: Rule
    services:
    - name: kubernetes-dashboard
      port: 80
    middlewares:
      - name: k8dashboard-https-redirect
```

### Proxy an external service

You first need to define endpoitns and service

```bash
kind: Endpoints
apiVersion: v1
metadata:
  name: truenas1
  namespace: truenas
subsets:
  - addresses:
    - ip: 10.0.50.21
    ports:
    - port: 80
      name: truenas1
---
apiVersion: v1
kind: Service
metadata:
  name: truenas1
  namespace: truenas
spec:
  ports:
     - protocol: TCP
       port: 80
       targetPort: 80
       name: truenas1
       nodePort: 0
```

Then you can point to the service as an internal service

## Kubernetes Dashboard

Install the dashboard:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.0/aio/deploy/recommended.yaml
```

### Creating sample user

In this guide, we will find out how to create a new user using the Service Account mechanism of Kubernetes, grant this user admin permissions and login to Dashboard using a bearer token tied to this user.

IMPORTANT: Make sure that you know what you are doing before proceeding. Granting admin privileges to Dashboard's Service Account might be a security risk.

For each of the following snippets for ServiceAccount and ClusterRoleBinding, you should copy them to new manifest files like dashboard-adminuser.yaml and use kubectl apply -f dashboard-adminuser.yaml to create them.

#### Creating a Service Account

We are creating Service Account with the name admin-user in namespace kubernetes-dashboard first.

```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

#### Creating a ClusterRoleBinding

In most cases after provisioning the cluster using kops, kubeadm or any other popular tool, the ClusterRole cluster-admin already exists in the cluster. We can use it and create only a ClusterRoleBinding for our ServiceAccount. If it does not exist then you need to create this role first and grant required privileges manually.

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

#### Getting a Bearer Token

Now we need to find the token we can use to log in. Execute the following command:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

It should print something like:

```bash
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXY1N253Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIwMzAzMjQzYy00MDQwLTRhNTgtOGE0Ny04NDllZTliYTc5YzEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.Z2JrQlitASVwWbc-s6deLRFVk5DWD3P_vjUFXsqVSY10pbjFLG4njoZwh8p3tLxnX_VBsr7_6bwxhWSYChp9hwxznemD5x5HLtjb16kI9Z7yFWLtohzkTwuFbqmQaMoget_nYcQBUC5fDmBHRfFvNKePh_vSSb2h_aYXa8GV5AcfPQpY7r461itme1EXHQJqv-SN-zUnguDguCTjD80pFZ_CmnSE1z9QdMHPB8hoB4V68gtswR1VLa6mSYdgPwCHauuOobojALSaMc3RH7MmFUumAgguhqAkX3Omqd3rJbYOMRuMjhANqd08piDC3aIabINX6gP5-Tuuw2svnV6NYQ
```

Now copy the token and paste it into the Enter token field on the login screen.

Click the Sign in button and that's it. You are now logged in as an admin.

## Metrics Server

Install the metrics server for autoscaling purposes.

First, download the high availability version of the yaml manifest to deploy two instances of metrics server in HA.

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability.yaml
```

Then, add the ```--kubelet-insecure-tls``` flag to the containers args:

```bash
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls
```

Also add the ```--enable-aggregator-routing=true``` flag to the apiserver configuration, so that requests sent to Metrics Server are load balanced between the 2 instances.

```bash
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

```bash
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.0.50.32
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://10.0.50.41:2379,https://10.0.50.42:2379,https://10.0.50.43:2379
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    - --enable-aggregator-routing=true
```

## Portainer

To deploy Portainer, get the yaml manifest:

wget <https://raw.githubusercontent.com/portainer/k8s/master/deploy/manifests/portainer/portainer.yaml>

Add a persistent volume and a persistent volume claim to the manifest just after the service account (needs to be after the namespace creation, before the actual deploy creation), and reference the claim in the volume definition:

```bash
---
# Source: portainer/templates/pvc.yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  namespace: portainer
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1
      - ip: 10.0.50.22
        hostname: truenas2
      - ip: 10.0.50.23
        hostname: truenas3
    ports:
      - port: 1
---
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster
  namespace: portainer
spec:
  ports:
  - port: 1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: portainer-pv
  namespace: portainer
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/portainer
    readOnly: false
  claimRef:
    name: portainer-pvc
    namespace: portainer
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: portainer-pvc
    namespace: portainer
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 10Gi
    storageClassName: ""
    volumeName: portainer-pv
```

```bash
      volumes:
        - name: "data"
          persistentVolumeClaim:
            claimName: portainer-pvc
```

Apply the manifest with kubectl

```bash
kubectl apply -f portainer.yaml
```

Go directly to the dashboard to create a user and login, or the instance will need to be restarted.

```bash
kubectl port-forward svc/portainer -n portainer 9000:9000
```

```bash
http://localhost:9000
```

Inside the dashboard, select the local environment, and go to to the Cluster/Setup page.

Here, define a "traefik" ingress class of type traefik, and enable "Enable features using the metrics API" (only if you installed the metrics server!). Save.

## kured

Get the yaml file to customize:

```bash
latest=$(curl -s https://api.github.com/repos/weaveworks/kured/releases | jq -r .[0].tag_name)
wget "https://github.com/weaveworks/kured/releases/download/$latest/kured-$latest-dockerhub.yaml"
```

Let's customize the following arguments, then apply the yaml:

```bash
Flags:
      --alert-filter-regexp regexp.Regexp   alert names to ignore when checking for active alerts
      --alert-firing-only bool              only consider firing alerts when checking for active alerts
      --blocking-pod-selector stringArray   label selector identifying pods whose presence should prevent reboots
      --drain-grace-period int              time in seconds given to each pod to terminate gracefully, if negative, the default value specified in the pod will be used (default: -1)
      --skip-wait-for-delete-timeout int    when seconds is greater than zero, skip waiting for the pods whose deletion timestamp is older than N seconds while draining a node (default: 0)
      --ds-name string                      name of daemonset on which to place lock (default "kured")
      --ds-namespace string                 namespace containing daemonset on which to place lock (default "kube-system")
      --end-time string                     schedule reboot only before this time of day (default "23:59:59")
      --force-reboot bool                   force a reboot even if the drain is still running (default: false)
      --drain-timeout duration              timeout after which the drain is aborted (default: 0, infinite time)
  -h, --help                                help for kured
      --lock-annotation string              annotation in which to record locking node (default "weave.works/kured-node-lock")
      --lock-release-delay duration         hold lock after reboot by this duration (default: 0, disabled)
      --lock-ttl duration                   expire lock annotation after this duration (default: 0, disabled)
      --message-template-uncordon string    message template used to notify about a node being successfully uncordoned (default "Node %s rebooted & uncordoned successfully!")
      --message-template-drain string       message template used to notify about a node being drained (default "Draining node %s")
      --message-template-reboot string      message template used to notify about a node being rebooted (default "Rebooting node %s")
      --notify-url                          url for reboot notifications (cannot use with --slack-hook-url flags)
      --period duration                     reboot check period (default 1h0m0s)
      --prefer-no-schedule-taint string     Taint name applied during pending node reboot (to prevent receiving additional pods from other rebooting nodes). Disabled by default. Set e.g. to "weave.works/kured-node-reboot" to enable tainting.
      --prometheus-url string               Prometheus instance to probe for active alerts
      --reboot-command string               command to run when a reboot is required by the sentinel (default "/sbin/systemctl reboot")
      --reboot-days strings                 schedule reboot on these days (default [su,mo,tu,we,th,fr,sa])
      --reboot-delay duration               add a delay after drain finishes but before the reboot command is issued (default 0, no time)
      --reboot-sentinel string              path to file whose existence signals need to reboot (default "/var/run/reboot-required")
      --reboot-sentinel-command string      command for which a successful run signals need to reboot (default ""). If non-empty, sentinel file will be ignored.
      --slack-channel string                slack channel for reboot notifications
      --slack-hook-url string               slack hook URL for reboot notifications [deprecated in favor of --notify-url]
      --slack-username string               slack username for reboot notifications (default "kured")
      --start-time string                   schedule reboot only after this time of day (default "0:00")
      --time-zone string                    use this timezone for schedule inputs (default "UTC")
      --log-format string                   log format specified as text or json, defaults to "text"
```

I will set the following arguments:

```bash
      --alert-filter-regexp=^RebootRequired$
      --end-time=17:00
      --notify-url=smtp://username:password@host:port/?fromAddress=fromAddress&toAddresses=recipient1
      --prometheus-url=http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local
      --reboot-days=mo,tu,we,th,fr
      --start-time=10:00
      --time-zone=Europe/Rome
```

Be aware to put your proper url to the prometheus service in the `--prometheus-url=` flag.

To check for prometheus alerts (not checking kured's alerts), reboot between 9 and 17 in weekdays, Europe/Rome timezone, send notifications by email.

```bash
kubectl apply -f "kured-$latest-dockerhub.yaml"
```
