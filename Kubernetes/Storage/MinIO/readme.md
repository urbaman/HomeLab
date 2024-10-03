# Install MinIO

Let's install the operator and a tenant

## Add the Helm Chart

```bash
helm repo add minio-operator https://operator.min.io
helm repo update
```

## Install the operator

**Note**: the console has been deprecated in the helm chart at the moment

```bash
helm show values minio-operator/operator > minio-operator-values.yaml
vi minio-operator-values.yaml
```

Let's keep the default values, but change them if you want

```bash
helm upgrade --install --namespace minio --create-namespace operator minio-operator/operator --values minio-operator-values.yaml
kubectl apply -f ig-minio-operator.yaml
```

**Note:** There might be a problem with the synchronization of the chart version and the docker image version, so you might need to install the previous version of the chart with `--version=version`, check it with `helm search repo minio-operator -l`

```bash
helm upgrade --install --namespace minio --create-namespace operator minio-operator/operator --version <old-chart-version>
```

### For the old helm chart with the console still working (!?)

```bash
kubectl apply -f ig-minio-operator.yaml
```

To access, go to the URL defined in the Traefik IngressRoute and insert the token obtained from the following command:

```bash
kubectl get secret/console-sa-secret -n minio -o json | jq -r ".data.token" | base64 -d
```

## Install the tenant

```bash
helm show values minio-operator/tenant > minio-tenant-values.yaml
```

Also prepare a `config.env` file to enable monitoring on the gui.

```bash
export MINIO_ROOT_USER=<user>
export MINIO_ROOT_PASSWORD=<password>
export MINIO_PROMETHEUS_URL=http://kube-prometheus-stack-prometheus.monitoring.svc.cluster.local:9090
export MINIO_PROMETHEUS_JOB_ID=minio
export MINIO_PROMETHEUS_AUTH_TYPE="public" # not needed
```

Create a secret from the file.

```bash
kubectl create secret generic minio-env-configuration -n minio     --from-file=./config.env
vi minio-tenant-values.yaml
```

Change the name, servers, pool name, volumes per server, volume size, storage class, enable the metrics, also enable the existingSecret specs with the secret name equal to the name of the secret you just created, and `prometheusOperator` to true.
Finally set serviceMetadata to add an `app: minio` label to the minio service (to target it with serviceMonitors):

```yaml
serviceMetadata:
    minioServiceLabels:
      app: minio
```

```bash
helm upgrade --install --namespace minio --create-namespace tenant minio-operator/tenant --values minio-tenant-values.yaml
kubectl apply -f ig-minio-tenant.yaml
```

**Note:** There might be a problem with the synchronization of the chart version and the docker image version, so you might need to install the previous version of the chart with `--version=version`, check it with `helm search repo minio-operator -l`

```bash
helm upgrade --install --namespace minio --create-namespace tenant minio-operator/tenant  --values minio-tenant.yaml --version <old-chart-version>
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
