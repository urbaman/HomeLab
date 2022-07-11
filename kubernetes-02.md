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
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
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

After implementing the prometheus-stack (see below), get the ptometheus-operator.yaml file from the github project, add the label ```release: prometheus``` to both the Pod Monitors and apply.

```bash
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: metallb-controller
  labels:
    release: prometheus
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
    release: prometheus
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

## Storage (GlusterFS)

### glusterfs-client

Fist you need to install the glusterfs-client package on your nodes. The client is used by the kubernetes scheduler to create the gluster volumes.

```bash
$ sudo apt install gluster-client
```

### Discovering GlusterFS in Kubernetes:

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

To persist the Gluster endpoints, you also need to create a relative service.

```bash
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster 
spec:
  ports:
  - port: 1
```

### Using GlusterFS in Kubernetes:

Create the desired path in the glusterfs volume (HDD5T/path/... in my situation)

##### Method 1 — Connecting to GlusterFS directly with Pod manifest:

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

##### Method 2 — Connecting using the PersistentVolume resource:

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

The create the relative PersistentVolumeClaim (be sure to name the PersistentVolume to Claim):

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
          name: mypd
    volumes:
      - name: mypd
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

To manage cloudflare DNS records we use cloudflare-operator from containeroo. It will check the ecternal public IP of the cluster and create/update selected records (A and CNAME) in all of our cloudflare account.

NOTE: backup any zone/domain/records you'll want to keep, we'll need to manage them through cloudflare-operator.

### Prerequisites

- Install Helm version 3 or later
- Install a supported version of Kubernetes

### Steps

#### Add the containeroo Helm repository (This repository is the only supported source of containeroo charts):

```bash
helm repo add containeroo https://charts.containeroo.ch
Update your local Helm chart repository cache:
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

TO add Cloudflare Operator to Prometheus/Graphana monitoring, get back here after ahbing implemented the stack, and create a podmonitor and a graphana ashboard

```bash
wget https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Edit the podmonitor yaml file to add the ```release: prometheus``` label

```
kubectl apply -f https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Download Grafana dashboard

```bash
wget https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/grafana/dashboards/overview.json -O /tmp/grafana-dashboard-cloudflare-operator.json
```

Create the configmap

```bash
kubectl create configmap grafana-dashboard-cloudflare-operator --from-file=/tmp/grafana-dashboard-cloudflare-operator.json
```

Add label so Grafana can fetch dashboard

```bash
kubectl label configmap grafana-dashboard-cloudflare-operator grafana_dashboard="1"
```

## Monitoring

Start checking the bind address of kube-proxy, kube-scheduler, kube-control-monitor

```bash
kubectl edit cm kube-proxy-config -n kube-system
## Change from
    metricsBindAddress: 127.0.0.1:10249 ### <--- Too secure
## Change to
    metricsBindAddress: 0.0.0.0:10249
```

And reload kube-proxy deployment

```bash
kubectl rollout restart deployment kube-proxy -n kube-system
```

On every control-panel, change bind adreess to 0.0.0.0:

```bash
sudo vi /etc/kubernetes/manifests/kube-scheduler.conf
sudo vi /etc/kubernetes/manifests/kube-control-manager.conf
```

The kill each scheduler and each control manager pods in the kube-system namespace

```bash
kubectl delete pod <pod-name> -n kube-system
```

Get the helm value file and add all your namespaces. Add others and upgrade the deploy if needed.

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

### Monitoring other services/apps on other namespaces

Take a look at the helm values of the prometheus-stack implementation:

```bash
kubectl get prometheus -n monitoring prometheus-kube-prometheus-prometheus -o yaml
```

Near the end you'll find something like:

```bash
  serviceMonitorSelector:
    matchLabels:
      release: prometheus
```

In here, ```release: prometheus``` is the label to add to the Service Monitors you'll need to create for the services to be monitored.

#### Example: Traefik (see more below for Traefik implementation)

Remmber to set the metrics port expose setting to true when installing Traefik, then create the following Service Monitor:

```bash
kind: ServiceMonitor
metadata:
  name: traefik-sm
  labels:
    traefik: http
    release: prometheus
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

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

https://adamtheautomator.com/prometheus-kubernetes/

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-kubernetes-monitoring-stack-with-prometheus-grafana-and-alertmanager-on-digitalocean#step-3-accessing-grafana-and-exploring-metrics-data

Exposing Grafana:

```bash
kubectl patch svc "$APP_INSTANCE_NAME-grafana" \
  --namespace "$NAMESPACE" \
  -p '{"spec": {"type": "LoadBalancer"}}'
```

## Traefik
https://www.padok.fr/en/blog/traefik-kubernetes-certmanager

https://www.howtogeek.com/devops/how-to-install-kubernetes-cert-manager-and-configure-lets-encrypt/

https://www.youtube.com/watch?v=dEAtD9PVr_Q

https://www.youtube.com/playlist?list=PL34sAs7_26wNldKrBBY_uagluNKC9cCak

https://www.youtube.com/watch?v=n5dpQLqOfqM

https://dev.to/bgalvao/traefik-lets-encrypt-cloudflare-36fj

### Create the traefik namespace

kubectl create namespace traefik

### Create the glusterfs endpoints in the traefik namespace

```bash
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

### Create the path in the gluster volume (from a node in the glusterfs cluster)

```bash
cd /cluster/HDD5T
mkdir k8sstorage
cd /k8sstorage
mkdir traefik-ssl
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

Go to the persistence section, turn it to true, uncomment the esistingClaim line and set it to the claim above.

Go to the ports section, and set to true both the metrics and the dashboard expose settings.

### Install

```bash
helm install traefik traefik/traefik --values traefik.yaml -n traefik
```

### Check the dashboard

Go to a machine with a browser and kubectl installed and properly configured, open the terminal

```bash
kubectl -n traefik port-forward traefik-667459fcbf-h7cqf 9000:9000
```

Now point the browser to http://localhost:9000/dashboard/ (remember the final slash)
