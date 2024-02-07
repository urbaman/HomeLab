# Install MinIO

Let's install the operator and a tenant

## Add the Helm Chart

```bash
helm repo add minio-operator https://operator.min.io
helm repo update
```

## Install the operator

```bash
helm show values minio-operator/operator > minio-operator.yaml
vi minio-operator.yaml
```

Let's keep the default values, but change them if you want

```bash
helm upgrade --install --namespace minio --create-namespace operator minio-operator/operator --values minio-operator.yaml
kubectl apply -f ig-minio-operator.yaml
```

**Note:** There might be a problem with the synchronization of the chart version and the docker image version, so you might need to install the previous version of the chart with `--version=version`, check it with `helm search repo minio-operator -l`

```bash
helm upgrade --install --namespace minio --create-namespace operator minio-operator/operator --version <old-chart-version>
kubectl apply -f ig-minio-operator.yaml
```

To access, go to the URL defined in the Traefik IngressRoute and insert the token obtained from the following command:

```bash
kubectl get secret/console-sa-secret -n minio -o json | jq -r ".data.token" | base64 -d
```

## Install the tenant

```bash
helm show values minio-operator/tenant > minio-tenant.yaml
vi minio-tenant.yaml
```

Change the user (accessKey) and password (secretKey), name, servers, pool name, volumes per server, volume size, storage class, enable the metrics then apply the chart.

```bash
helm upgrade --install --namespace minio --create-namespace tenant minio-operator/tenant --values minio-tenant.yaml
kubectl apply -f ig-minio-tenant.yaml
```

**Note:** There might be a problem with the synchronization of the chart version and the docker image version, so you might need to install the previous version of the chart with `--version=version`, check it with `helm search repo minio-operator -l`

```bash
helm upgrade --install --namespace minio --create-namespace tenant minio-operator/tenant --version <old-chart-version> -set existingSecret.name=TENANT-NAME-env-configuration --set tenant.name=TENANT-NAME --set tenant.configuration.name=TENANT-NAME-env-configuration --set tenant.pools.servers=9 --set tenant.pools.name=pool-0 --set tenant.pools.volumesPerServer=4 --set tenant.pools.size=10Gi --set tenant.pools.storageClassName=rook-ceph-nvme2tb
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
