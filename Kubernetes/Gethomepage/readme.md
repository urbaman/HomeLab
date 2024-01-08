# Gethomepage installation

![Screenshot](https://github.com/urbaman/HomeLab/blob/main/Kubernetes/Gethomepage/images/gethomepage.png?raw=true)

## Install through helm chart

```bash
helm repo add jameswynn https://jameswynn.github.io/helm-charts
helm repo update
helm show values jameswynn/homepage > gethomepage-values.yaml
```

Set `enableRbac: true`, `serviceAccount.create: true`, `kubernetes.mode: cluster`, `repository: ghcr.io/gethomepage/homepage`, `tag: latest` then install

```bash
helm upgrade --install gethomepage --create-namespace --namespace=gethomepage jameswynn/homepage -f gethomepage-values.yaml
```

Eventually, delete and recreate the configmap with your settings:

```bash
kubectl delete cm -n gethomepage gethomepage
kubectl apply -f gethomepage-cm.yaml
```

To save your settings:

```bash
kubectl get cm -n gethomepage gethomepage -o yaml > gethomepage-cm.yaml
```

## Update the version and add proper dns resolving

```bash
kubectl edit -n gethomepage deployment gethomepage
```

```bash
      - image: ghcr.io/gethomepage/homepage:latest
```

After the volumemounts

```bash
      dnsConfig:
        options:
          - name: ndots
            value: "1"
```

## Apply the traefik ingress route

Set the username and password in the basic-auth secret, and your domain.

```bash
kubectl apply -f ig-gethomepage.yaml
```
