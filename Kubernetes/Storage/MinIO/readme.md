# Install MinIO

Let's install the operator and a tenant

## Add the Helm Chart

```bash
helm repo add minio-operator https://operator.min.io
helm repo update
```

## Install the operator

```bash
helm repo show values minio-operator/operator > minio-operator.yaml
vi minio-operator.yaml
```

Let's keep the default values, but change them if you want

```bash
helm install --namespace minio-operator --create-namespace operator minio-operator/operator --values minio-operator.yaml
kubectl apply -f ig-minio-operator.yaml
```

To access, go to the URL defined in the Traefik IngressRoute and insert the token obtained from the following command:

```bash
kubectl get secret/console-sa-secret -n minio-operator -o json | jq -r ".data.token" | base64 -d
```

## Install the tenant

```bash
helm repo show values minio-operator/tenant > minio-tenant.yaml
vi minio-tenant.yaml
```

Change the user (accessKey) and password (secretKey), name, servers, pool name, volumes per server, volume size, storage class, then apply the chart.

```bash
helm install --namespace MINIO_TENANT_NAMESPACE --create-namespace MINIO_TENANT_NAME minio-operator/tenant --values minio-tenant.yaml
kubectl apply -f ig-minio-tenant.yaml
```

To access, go to the URL defined in the Traefik IngressRoute, and use accessKey/secretKey.

To check the s3-like API, use the mc command:

```bash
sudo curl https://dl.min.io/client/mc/release/linux-amd64/mc --create-dirs -o /usr/bin/mc
sudo chmod +x /usr/bin/mc
mc --help
```

Press ESC to exit the help, then set an alias and check the API:

```bash
mc alias set ALIAS_NAME HOSTNAME ACCESS_KEY SECRET_KEY
mc admin info myminio
```
