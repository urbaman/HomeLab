# Velero for S3 Backup

## Create a bucket and an access key

For both Minio and Contabo (or any other S3-compatible object storage API), create a bucket and an access key.

### Secret for CLI Installation (easy one backup location)

Crate a file `credentials-velero` with the access key

```bash
[default]
aws_access_key_id = <KEY>
aws_secret_access_key = <SECRET>
```

### Secret for Helm Installation (easy multiple Backup Locations)

Crate file(s) `veleroMinioSecret` and `veleroContaboSecret` with the access key(s)

```bash
[default]
aws_access_key_id = <KEY>
aws_secret_access_key = <SECRET>
```

## Install via CLI (easy one backup location)

### Minio example

```bash
velero install \
--provider aws \
--plugins velero/velero-plugin-for-aws:v1.9.0 \
--bucket velero-test  \
--secret-file ./credentials-velero  \
--default-volumes-to-fs-backup \
--use-node-agent \
--use-volume-snapshots=false \
--backup-location-config region=default,s3ForcePathStyle="true",s3Url=https://minio-s3.urbaman.it  \
--image velero/velero:v1.13.0
```

### Contabo example

```bash
velero install \
--provider aws \
--plugins velero/velero-plugin-for-aws:v1.9.0 \
--bucket mybucket \
--use-volume-snapshots=false \
--default-volumes-to-fs-backup \
--secret-file ./credentials-velero \
--backup-location-config region=eu,s3ForcePathStyle="true",s3Url=https://eu2.contabostorage.com \
--image velero/velero:v1.13.0
```

## Install via Helm (easy multiple Backup Locations)

Create namespace and secrets

```bash
kubectl create ns velero
kubectl create secret generic -n velero velero-minio-secret --from-file=config=./veleroMinioSecret
kubectl create secret generic -n velero velero-contabo-secret --from-file=config=./veleroContaboSecret
```

Add the repo, set the values for backup locatiuons, set nodeAgent to true and snapshots to false (note: probably doable with nfs-csi instead of nfs-provisioner)

```bash
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm repo update
helm show values vmware-tanzu/velero > velero-values.yaml
```

Install or upgrade

```bash
helm upgrade -i -n velero --create-namespace velero vmware-tanzu/velero -f velero-values.yaml
```

## Backup

Single Storage Location

```bash
velero backup create test-nginx-b4 --include-namespaces test-nginx --wait
```

Multiple Storage Location

```bash
velero backup create <storage-location>-test-nginx-b4 --include-namespaces test-nginx --storage-location <storage-location> --wait
```

## Restore

Single Storage Location

```bash
velero restore create --from-backup test-nginx-b4
```

Multiple Storage Location

```bash
velero restore create --from-backup <storage-location>-test-nginx-b4
```

## Delete

```bash
velero backup delete test-nginx-b4
```

## Schedule

Single Storage Location

```bash
velero schedule create test-nginx-b4 --include-namespaces test-nginx --schedule "0 7 * * *"
```

Multiple Storage Location

```bash
velero schedule create <storage-location>-test-nginx-b4 --include-namespaces test-nginx --storage-location <storage-location> --schedule "0 7 * * *"
```

## Retention

Usually 30d, set `--ttl <DURATION>` to specify

## UI

See Cloudcasa, register and deploy the agent
