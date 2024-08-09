# Kubetail installation

```bash
helm repo add kubetail https://kubetail-org.github.io/helm-charts/
helm repo update
helm upgrade -i kubetail kubetail/kubetail --namespace kubetail --create-namespace
kubectl apply -f ig-kubetail.yaml
```
