# Cilium networking

We will see how to install Cilium as the base cni for our cluster.

Things we will install/get:

- Kubeproxy replacement
- eBPF datapath
- Loadbalancing (no need for Metallb) with L2 Announcements
- Ingress controller in shared mode (just one ingress point)
- Gateway API (alternative to ingress)
- Network encryption
- Hubble UI for Network, Service & Security Observability

We will start adding the helm chart, and creating our values file to customize:

```bash
helm repo add cilium https://helm.cilium.io/
helm show values cilium/cilium > cilium-values.yaml 
```

We then customize the values as per the example (see the cilium documentation for specifics and other settings)

Before installing the chart, we need to create a secret with the keys for the network encryption and the CRDs for the Gateway API

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/standard/gateway.networking.k8s.io_gatewayclasses.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/standard/gateway.networking.k8s.io_gateways.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/standard/gateway.networking.k8s.io_httproutes.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/standard/gateway.networking.k8s.io_referencegrants.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/standard/gateway.networking.k8s.io_grpcroutes.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/gateway-api/v1.2.0/config/crd/experimental/gateway.networking.k8s.io_tlsroutes.yaml
kubectl create -n kube-system secret generic cilium-ipsec-keys     --from-literal=keys="3+ rfc4106(gcm(aes)) $(dd if=/dev/urandom count=20 bs=1 2> /dev/null | xxd -p -c 64) 128"
```

Then we can install the chart

```bash
helm upgrade -i cilium cilium/cilium --namespace kube-system -f cilium-values.yaml
```

Finally, with the chart installed and the pods in ready state, we create the loadbalancer Policy and IP Pool

```bash
kubectl apply -f cilium-loadbalancer.yaml
```
