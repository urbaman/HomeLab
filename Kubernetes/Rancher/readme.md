# Deploy Rancher K8s Dashboard

Add the repo, create a values file and deploy in the cattle-system namespace (needed!)

```bash
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cattle-system
helm show values rancher-latest/rancher > rancher-values.yaml
vi rancher-values.yaml
helm upgrade -i rancher -n cattle-system rancher-latest/rancher -f rancher-values.yaml
kubectl apply -f ig-rancher.yaml
```

You can install the kube-prometheus-stack from rancher (the monitoring app), following the helm values configurations from the [Prometheus-Stack section](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Prometheus-Stack).

Remember to unset the resource limits and requests, to keep the stack working and not going OOM.
