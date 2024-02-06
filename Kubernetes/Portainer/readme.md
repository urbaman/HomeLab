# Portainer

To deploy Portainer, add the helm repo

```bash
helm repo add portainer https://portainer.github.io/k8s/
helm repo update
helm show values portainer/portainer > portainer-values.yaml
```

Set the service.type to ClusterIP, the tls.force=true and the persistence.storageclass to rook-ceph-nvme2tb (or whatever storageclass you're using)

```bash
helm upgrade -i portainer portainer/portainer --create-namespace --namespace portainer -f portainer-values.yaml
```

Or set the values directly

```bash
helm upgrade -i portainer portainer/portainer --create-namespace --namespace portainer --set service.type=ClusterIP --set tls.force=true --set persistence.storageclass=rook-ceph-nvme2tb
```

Go directly to the dashboard to create a user and login, or the instance will need to be restarted.

```bash
kubectl port-forward svc/portainer -n portainer 9443
```

```bash
http://localhost:9443
```

Inside the dashboard, select the local environment, and go to to the Cluster/Setup page.

Here, define a "traefik" ingress class of type traefik, and enable "Enable features using the metrics API" (only if you installed the metrics server!). Save.

## Traefik exposure (admin portaineradmin)

```bash
kubectl apply -f ig-portainer.yaml
```
