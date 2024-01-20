# Nextcloud Monitoring

Finally, we deploy the nextcloud exporter for monitoring, but we need another step before that: creating an auth token for nextcloud, using its occ function.

## Create the Credentials

Option 1: not working, create a token

```bash
kubectl exec -it -n nextcloud <nextcloud_pod_name> -- bash
TOKEN=$(openssl rand -hex 32)
su -s /bin/bash www-data -c "php occ config:app:set serverinfo token --value "$TOKEN""
```

Option 2: create a user admin, disable the MFA for him, use username and password as credentials.

## Monitoring Deploy

And then, deploy the `sm-nextcloud.yaml` file, changing the auth-token or the username and password with the one just generated, commenting the variables not used.

```bash
kubectl apply -f sm-nextcloud.yaml
```

Last step: the grafana dashboard.

```bash
kubectl create configmap grafana-dashboard-nextcloud -n monitoring --from-file=nextcloud-grafana.json
kubectl label configmap grafana-dashboard-nextcloud -n monitoring grafana_dashboard="1"
```
