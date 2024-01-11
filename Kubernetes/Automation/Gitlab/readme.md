# Gitlab Installation

Always check the [documentation](https://docs.gitlab.com/charts/).

## Prerequisites and settings

### PostgreSQL

Have a postgreSQL server up and running (see [Postgresql replica cluster with Pgadmin](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Postgresql)).

Create a dedicated user and a dedicated DB (gitlab, gitlab), the user must be owner.

Create a secret containing the user password (gitlab-postgresql-password, postgres-password).

Set the helm values:

```yaml
postgresql.install=false
global.psql.host=psql.example
global.psql.password.secret=gitlab-postgresql-password
global.psql.password.key=postgres-password
global.psql.port
global.psql.database
global.psql.username
```

### Redis

Have a Redis server up and running (see [Redis replica cluster](https://github.com/urbaman/HomeLab/tree/main/Kubernetes/Database/Redis)).

Create a secret with the Redis password (gitlab-redis, redis-password) or point to the original redis secret (probably you can't).

Set the helm values:

```yaml
redis.install=false
global.redis.host=redis.example
global.redis.auth.enabled=true
global.redis.auth.secret=gitlab-redis
global.redis.auth.key=redis-password
```

### Minio

Have a Minio tenant up and running.

Create the following buckets:

- gitlab-registry-storage
- gitlab-lfs-storage
- gitlab-artifacts-storage
- gitlab-uploads-storage
- gitlab-packages-storage
- gitlab-backup-storage
- gitlab-tmp-storage

Create a secret with the Minio connection specs (object-storage, config) for all of the buckets but registry and backup.

```yaml
provider: AWS
# Specify the region
region: us-east-1
# Specify access/secret keys
aws_access_key_id: AWS_ACCESS_KEY
aws_secret_access_key: AWS_SECRET_KEY
# The below settings are for S3 compatible endpoints
#   See https://docs.gitlab.com/ee/administration/object_storage.html#s3-compatible-connection-settings
# aws_signature_version: 4
# host: storage.example.com
endpoint: "https://minio.example.com:9000"
path_style: true
```

Create a secret with the Minio connection specs (registry-storage, config) for registry.

```yaml
s3:
  bucket: gitlab-registry-storage
  accesskey: AWS_ACCESS_KEY
  secretkey: AWS_SECRET_KEY
  region: us-east-1
  regionendpoint: "https://minio.example.com:9000"
  v4auth: true
```

**Note:** The bucket name needs to be set both in the secret, and in global.registry.bucket. The secret is used in the registry server, and the global is used by GitLab backups.

Create a secret with the Minio connection specs (storage-config, config) for backups.

```yaml
[default]
access_key = AWS_ACCESS_KEY
secret_key = AWS_SECRET_KEY
bucket_location = us-east-1
multipart_chunk_size_mb = 128 # default is 15 (MB)
```

Define the helm values:

```yaml
global:
  minio:
    enabled: false
  registry:
    bucket: gitlab-registry-storage
  appConfig:
    lfs:
      bucket: gitlab-lfs-storage
      connection:
        secret: object-storage
        key: connection
    artifacts:
      bucket: gitlab-artifacts-storage
      connection:
        secret: object-storage
        key: connection
    uploads:
      bucket: gitlab-uploads-storage
      connection:
        secret: object-storage
        key: connection
    packages:
      bucket: gitlab-packages-storage
      connection:
        secret: object-storage
        key: connection
    externalDiffs:
      bucket: gitlab-externalDiffs-storage
      connection:
        secret: object-storage
        key: connection
    terraformState:
      bucket: gitlab-terraformState-storage
      connection:
        secret: object-storage
        key: connection
    dependencyProxy:
      bucket: gitlab-dependencyProxy-storage
      connection:
        secret: object-storage
        key: connection
    ciSecureFiles:
      bucket: gitlab-ciSecureFiles-storage
      connection:
        secret: object-storage
        key: connection
    backups:
      bucket: gitlab-backup-storage
      tmpBucket: gitlab-tmp-storage
gitlab:
  toolbox:
    backups:
      objectStorage:
        config:
          secret: s3cmd-config
          key: config
registry:
  storage:
    secret: registry-storage
    key: config
```

Various things to do (external prostgres, redis, minio, cert-manager, traefik, prometheus)

## Cert-manager and Traefik

Create an entrypoint for the Gitlab Shell on Traeifk (port 2232?).

Create certificates with cert-manager (gitlab, registry, ?)

```yaml
certmanager.install=false
global.ingress.configureCertmanager=false
gitlab.webservice.ingress.tls.secretName=urbaman-gitlab
registry.ingress.tls.secretName=urbaman-gl-gitlab
```

```yaml
global:
  ingress:
    # Default, present here to be explicit.
    enabled: false
  shell:
    port: 2232
    tcp: # Probably needed for Traefik?
      proxyProtocol: true # Probably needed for Traefik? default false

nginx-ingress:
  # Disable the deployment of the in-chart NGINX Ingress provider.
  enabled: false
```

Make IngressRoute for gitlab and registry, make IngressRouteTCP for githlab-shell.

```yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRouteTCP
metadata:
  name: gitlab-shell
  namespace: gitlab
  labels:
spec:
  entryPoints:
    - gitlab-shell
  routes:
  - match: HostSNI(`*`)
    services:
    - name: gitlab-shell # Put the gitlab-shell service name
      namespace: gitlab
      port: 2232 # Put gitlab.shell.port
      proxyProtocol: # Only if global.shell.tcp.proxyProtocol
        version: 2  # Only if global.shell.tcp.proxyProtocol
```

## Persistence

Check the subcharts.

```yaml
gitaly:
  persistence:
    enabled: true
    size: 50Gi
    storageClass: ""
```

## Prometheus

Add the `gitlab` namespace to the prometheus operator deployment config.

Disable the internal prometheus.

```yaml
prometheus:
  install: false
```

Enable all the subcharts metrics and servicemonitors.

```yaml
gitaly:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

gitlab-exporter:
  enabled: true
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

gitlab-shell:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

kas:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

migrations:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

sidekiq:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack

webservice:
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true
      additionalLabel:
        release: kube-prometheus-stack
```

## Installation

```bash
helm repo add gitlab https://charts.gitlab.io/
helm repo update
helm upgrade --install gitlab gitlab/gitlab -n gitlab --create-namespace --values gitlab-values.yaml
```

## Post-installation

Login with user `root` and the following secret:

```bash
kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
```

## Upgrade

Follow the instructions: [Upgrade the GitLab chart](https://docs.gitlab.com/charts/installation/upgrade.html#upgrade-to-version-70)
