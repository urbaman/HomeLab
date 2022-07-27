# Nextcloud

...

## Monitoring

...

### Create an Auth Token

```bash
kubectl exec -it -n nextcloud <nextcloud_pod_name> -- bash
TOKEN=$(openssl rand -hex 32)
su -s /bin/bash www-data -c "php occ config:app:set serverinfo token --value "$TOKEN""
```

...
