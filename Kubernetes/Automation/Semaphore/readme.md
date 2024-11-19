# Semaphore

## External behind Traefik

```bash
kubectl apply -f ig-semaphore.yaml
```

## Deploy in the Cluster

Create a MySQL/MariaDB db and user (semaphoreui/semaphoreui)

Create a key (for the SEMAPHORE_ACCESS_KEY_ENCRYPTION environment variable)

```bash
head -c32 /dev/urandom | base64
```

Encode the passwords:

```bash
echo -n '<SEMAPHORE_DB_PASS>' | base64
echo -n '<SEMAPHORE_ADMIN_PASSWORD>' | base64
echo -n '<SEMAPHORE_EMAIL_PASSWORD>' | base64
```

Put them in the secret, then apply the deployment

```bash
kubectl apply -f semaphoreui.yaml
```
