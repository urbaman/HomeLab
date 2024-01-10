# Gitlab Installation

Various things to do (external prostgres, redis, minio, cert-manager, traefik, prometheus)

## Redis

redis.install=false
global.redis.host=redis.example
global.redis.auth.enabled=false

## postgres

postgresql.install=false
global.psql.host=psql.example
global.psql.password.secret=gitlab-postgresql-password
global.psql.password.key=postgres-password
global.psql.port
global.psql.database
global.psql.username

## minio

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
        secret: objectstore-lfs
        key: connection
    artifacts:
      bucket: gitlab-artifacts-storage
      connection:
        secret: objectstore-artifacts
        key: connection
    uploads:
      bucket: gitlab-uploads-storage
      connection:
        secret: objectstore-uploads
        key: connection
    packages:
      bucket: gitlab-packages-storage
      connection:
        secret: objectstore-packages
        key: connection
    externalDiffs:
    terraformState:
    dependencyProxy:
    ciSecureFiles:
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

### Connection secret

[S3 Compatible](https://docs.gitlab.com/ee/administration/object_storage.html#amazon-s3)
[S3 Compatible for backups](https://docs.gitlab.com/charts/advanced/external-object-storage/index.html#backups-storage-example)

## Cert-manager and Traefik

certmanager.install=false
global.ingress.configureCertmanager=false
gitlab.webservice.ingress.tls.secretName=RELEASE-gitlab-tls
registry.ingress.tls.secretName=RELEASE-registry-tls
minio.ingress.tls.secretName=RELEASE-minio-tls
gitlab.kas.ingress.tls.secretName=RELEASE-kas-tls

```yaml
global:
  ingress:
    # Default, present here to be explicit.
    enabled: true
    # Toggle the TCP configuration and annotations to Traefik.
    provider: traefik
    # Alter the `kubernetes.io/ingress.class` annotation or
    # `spec.ingressClassName` value chart-wide.
    class: traefik
    annotations:
      # Tell Traefik that we've configured TLS
      # NOTE: disable this if `global.ingress.tls.enabled=false`.
      traefik.ingress.kubernetes.io/router.tls: "true"
      # Ensure the HTTP Routes only listen on 443, rather than all entrypoints.
      # NOTE: set the value to `web` if `global.ingress.tls.enabled=false`.
      traefik.ingress.kubernetes.io/router.entrypoints: websecure
  # Tell the gitlab-shell chart which traefik entrypoint to use
  # Default, present here to be explicit. 
  gitlab-shell:
    traefik:
      entrypoint: "gitlab-shell"

nginx-ingress:
  # Disable the deployment of the in-chart NGINX Ingress provider.
  enabled: false
```

## Prometheus

prometheus.install=true

Enable the Gitlab exporter subchart and all the other subcharts metrics and servicemonitors
