# Portainer

To deploy Portainer, add the helm repo

```bash
helm repo add portainer https://portainer.github.io/k8s/
helm repo update
helm show values portainer/portainer > portanier-values.yaml
```

Set the service.type to CLusterIP, the tls.force=true and the persistence.storageclass to longhorn (or whatever storageclass you're using)

```bash
helm install portainer portainer/portainer --create-namespace --namespace portainer -f portainer-values.yaml
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
kubectl apply -f portainer-traefik.yaml
```
