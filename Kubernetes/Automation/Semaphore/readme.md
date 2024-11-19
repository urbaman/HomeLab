# Semaphore

## External behind Traefik

```bash
kubectl apply -f ig-semaphore.yaml
```

## Deploy in the Cluster

Create a MySQL/MariaDB db and user (semaphore/semaphore)

Create a key (for the SEMAPHORE_ACCESS_KEY_ENCRYPTION environment variable)

```bash
head -c32 /dev/urandom | base64
```

Deploy

docker run --name semaphore \
-p 3000:3000 \
-e SEMAPHORE_DB_DIALECT=mysql \
-e SEMAPHORE_DB_HOST=mysql.mysql.svc.cluster.local \
-e SEMAPHORE_DB_NAME=semaphoreui \
-e SEMAPHORE_DB_USER=semaphoreui \
-e SEMAPHORE_DB_PASS=db_password \
-e SEMAPHORE_ADMIN=admin \
-e SEMAPHORE_ADMIN_PASSWORD=semaphoreuiadmin \
-e SEMAPHORE_ADMIN_NAME="Admin" \
-e SEMAPHORE_ADMIN_EMAIL=admin@urbaman.it \
-v semaphore_data:/var/lib/semaphore \
-v semaphore_config:/etc/semaphore \
--network semaphore_network \
-e SEMAPHORE_EMAIL_ALERT=true \
-e SEMAPHORE_EMAIL_SENDER=semaphoreui@urbaman.it \
-e SEMAPHORE_EMAIL_HOST=mail.urbaman.it \
-e SEMAPHORE_EMAIL_PORT=465 \
-e SEMAPHORE_EMAIL_USERNAME=k8sadmin@urbaman.it \
-e SEMAPHORE_EMAIL_PASSWORD=gGrbS1Vw6ha60juMe \
-e SEMAPHORE_EMAIL_SECURE=true \
-d semaphoreui/semaphore:v2.10.35
