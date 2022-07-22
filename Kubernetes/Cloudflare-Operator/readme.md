# Cloudflare DynDNS and DNS records

To manage cloudflare DNS records we use cloudflare-operator from containeroo. It will check the ecternal public IP of the cluster and create/update selected records (A and CNAME) in all of our cloudflare account.

NOTE: backup any zone/domain/records you'll want to keep, we'll need to manage them through cloudflare-operator.

## Prerequisites

- Install Helm version 3 or later
- Install a supported version of Kubernetes

## Steps

### Add the containeroo Helm repository (This repository is the only supported source of containeroo charts):

```bash
helm repo add containeroo https://charts.containeroo.ch
```

Update your local Helm chart repository cache:

```bash
helm repo update
```

### Install CustomResourceDefinitions

cloudflare-operator requires a number of CRD resources, which must be installed manually using kubectl.

```bash
kubectl apply -f https://github.com/containeroo/cloudflare-operator/releases/download/v0.2.3/crds.yaml
```

### Install cloudflare-operator

To install the cloudflare-operator Helm chart, use the Helm install command as described below (check last version)

```bash
helm install \
  cloudflare-operator containeroo/cloudflare-operator \
  --namespace cloudflare-operator \
  --create-namespace \
  --version v0.2.4
```

## Preparation

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

## Examples

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

## Monitoring

TO add Cloudflare Operator to Prometheus/Graphana monitoring, get back here after having implemented the stack, and create a podmonitor and a grafana dashboard

```bash
wget https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Edit the podmonitor yaml file to add the ```release: kube-prometheus-stack``` label

```
kubectl apply -f https://raw.githubusercontent.com/containeroo/cloudflare-operator/master/config/manifests/prometheus/monitor.yaml
```

Install the dashboard in grafana, download the json and create a configmap fron it in the monitoring namespace.

```bash
kubectl create configmap grafana-dashboard-cloudflare-operator -n monitoring --from-file=/tmp/grafana-dashboard-cloudflare-operator.json
```

Add label so Grafana can fetch the dashboard

```bash
kubectl label configmap grafana-dashboard-cloudflare-operator -n monitoring grafana_dashboard="1"
```