# Install Headlamp dashboard

## install with helm

```bash
helm repo add headlamp https://kubernetes-sigs.github.io/headlamp/
helm repo update
helm show values headlamp/headlamp > headlamp-values.yaml
vi headlamp-values.yaml
```

Change the values to your liking (see example, it conmtains the initContainer for the kubescape plugin) and deploy, then add the traefik ingressRoute

```bash
helm upgrade -i headlamp headlamp/headlamp --namespace kube-system -f headlamp-values.yaml
kubectl apply -f ig-headlamp.yaml
```
