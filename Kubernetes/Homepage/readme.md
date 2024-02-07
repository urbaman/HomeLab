# homepage installation

![Screenshot](https://github.com/urbaman/HomeLab/blob/main/Kubernetes/Homepage/images/homepage.png?raw=true)

## Install through helm chart

```bash
helm repo add jameswynn https://jameswynn.github.io/helm-charts
helm repo update
helm show values jameswynn/homepage > homepage-values.yaml
```

Set `enableRbac: true`, `serviceAccount.create: true`, `kubernetes.mode: cluster`, `repository: ghcr.io/homepage/homepage`, `tag: latest` then install

```bash
helm upgrade --install homepage --create-namespace --namespace=homepage jameswynn/homepage -f homepage-values.yaml
```

Eventually, delete and recreate the configmap with your settings:

```bash
kubectl delete cm -n homepage homepage
kubectl apply -f homepage-cm.yaml
```

To save your settings:

```bash
kubectl get cm -n homepage homepage -o yaml > homepage-cm.yaml
```

## Update the version

```bash
kubectl edit -n homepage deployment homepage
```

```yaml
      - image: ghcr.io/homepage/homepage:latest
```

## Apply the traefik ingress route

Set the username and password in the basic-auth secret, and your domain.

```bash
kubectl apply -f ig-homepage.yaml
```
