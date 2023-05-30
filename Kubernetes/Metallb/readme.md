# Metallb

## Preparation

If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

You can achieve this by editing kube-proxy config in current cluster:

```bash
kubectl edit configmap -n kube-system kube-proxy
```

and set:

```yaml
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

## Installation By Manifest

To install MetalLB, apply the manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.9/config/manifests/metallb-native.yaml
```

## Configuring L2 configuration

Layer 2 mode is the simplest to configure: in many cases, you don’t need any protocol-specific configuration, only IP addresses.

Layer 2 mode does not require the IPs to be bound to the network interfaces of your worker nodes. It works by responding to ARP requests on your local network directly, to give the machine’s MAC address to clients.

In order to advertise the IP coming from an IPAddressPool, an L2Advertisement instance must be associated to the IPAddressPool.

For example, the following configuration gives MetalLB control over IPs from 10.0.50.100 to 10.0.50.199 (free ip range on the same network of the k8s cluster hosts, so that che services will be accessible on that network), and configures Layer 2 mode:

```yaml
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

```yaml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: metallb-l2-advertisement
  namespace: metallb-system
spec:
  ipAddressPools:
  - metallb-pool
```

## Monitoring

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