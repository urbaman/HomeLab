# Semaphore

## External behind Traefik

```bash
kubectl apply -f ig-semaphore.yaml
```

## Deploy in the Cluster

### With a deployment

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

### Via Helm

Create a postgresql DB, a namespace and some secrets, then add the repo and customize the values

```bash
kubectl create namespace semaphore
kubectl create secret generic -n semaphore semaphore-database-secret --from-literal=username=semaphore --from-literal=password=<db-password>
kubectl create secret generic -n semaphore semaphore-email-secret --from-literal=username=smtp@domain.com --from-literal=password=<smtp-password>
kubectl create secret generic -n semaphore semaphore-admin-secret --from-literal=username=<admin-username> --from-literal=password=<admin-password> --from-literal=fullname="Semaphore Admin" --from-literal=email=mail@domain.com
helm repo add semaphoreui https://semaphoreui.github.io/charts
helm repo update
helm show values semaphoreui/semaphore > semaphore-values.yaml
```

Finally, install the helm and deploy the ingressRoute

```bash
helm upgrade -i -n semaphore --create-namespace semaphore semaphoreui/semaphore -f semaphore-values.yaml
kubectl apply -f ig-semaphore.yaml
```
