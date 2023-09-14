# Postgresql cluster

## Installation

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install postgresql bitnami/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set persistence.storageClass=longhorn --set persistence.size=15Gi --set persistence.accessModes={"ReadWriteMany}
```

## Connection

Get the password for the posgres (admin) user

```bash
kubectl get secret -n postgresql postgresql-postgresql-ha-postgresql -o jsonpath='{.data.\password}' | base64 -d
```

Access the postgresql-postgresql-ha-pgpool service on port 5432 (eventually through kubectl proxy) with user postgres and just found password

```bash
kubectl port-forward -n postgresql service/postgresql-postgresql-ha-pgpool :5432
```

You can also use the service dns inside the cluster: postgresql-postgresql-ha-pgpool.postgresql.svc.cluster.local

## Pgadmin

Change the password in the secret, change the domains in the certificate and ingressroutes, then apply the yaml.

```bash
kubectl apply -n pgadmin.yaml
```
