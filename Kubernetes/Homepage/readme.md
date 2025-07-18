# homepage installation

![Screenshot](https://github.com/urbaman/HomeLab/blob/main/Kubernetes/Homepage/images/homepage.png?raw=true)

## Install through manifests

```bash
kubectl apply -f homepage.yaml
```

## Manage the configuration

Eventually, delete and recreate the configmap with your settings:

```bash
vi homepage.yaml
kubectl apply -f homepage-cm.yaml
kubectl rollout restart deployment -n homepage homepage
```
