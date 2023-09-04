# Gethomepage installation

![Screenshot](https://github.com/urbaman/HomeLab/blob/main/Kubernetes/Gethomepage/images/gethomepage.png?raw=true)

## Install through helm chart

```bash
helm repo add jameswynn https://jameswynn.github.io/helm-charts
helm show values jameswynn/homepage > gethomepage-values.yaml
helm upgrade --install gethomepage --create-namespace --namespace=gethomepage jameswynn/homepage -f gethomepage-values.yaml
```

## Apply the traefik ingress route

```bash
kubectl apply traefik-gethomepage.yaml
```
