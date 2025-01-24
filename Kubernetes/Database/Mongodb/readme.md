# MongoDB installation

## Prerequisites

You need the AVX feature enabled for your CPU

```bash
cat /proc/cpuinfo | grep avx
```

## Prepare

Get the helm values and customize

```bash
helm show values oci://registry-1.docker.io/bitnamicharts/mongodb > mongodb-values.yaml
vi mongodb-values.yaml
```

Set persistence, resourcesPreset, metrics and servicemonitor values.

```yaml
persistence:
  storageClass: "ceph-rbd"
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    labels: kube-prometheus-stack
resourcesPreset: "small"
```

## Install

```bash
helm upgrade -i mongodb -n mongodb --create-namespace oci://registry-1.docker.io/bitnamicharts/mongodb -f mongodb-values.yaml
```

## Connect

Get the root password, and use it to connect with the root user

```bash
kubectl get secret --namespace mongodb mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d
```

## Upgrade

```bash
helm upgrade -i mongodb -n mongodb --create-namespace oci://registry-1.docker.io/bitnamicharts/mongodb -f mongodb-values.yaml --set auth.rootPassword=<password>
```
