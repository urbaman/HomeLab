# ArgoCD deployment

## Prerequisites

Create a `argocd` namespace.

Have a Redis server up and running.

Create a secret `argocd-redis-auth` in the `argocd` namespace with the redis password in the `redis-password` key.

Create a `argocd-vaules.yaml` file (see example for HA with autoscaling), setting the redis values for host and secret.

Add the `argocd` namespace in the prometheus operator deployment config.

Review the traefik settings in the `ig-argocd.yaml` file.

## Installation

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm upgrade -i argocd argo/argo-cd -n argocd --create-namespace --values argocd-values.yaml
kubectl apply -f ig-argocd.yaml
```

As shown in the helm output, you can access ArgoCD with username admin and password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
