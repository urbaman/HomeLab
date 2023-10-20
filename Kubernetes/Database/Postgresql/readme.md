# Postgresql cluster

## Installation

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade -i postgresql oci://registry-1.docker.io/bitnamicharts/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set persistence.storageClass=longhorn-nvme --set persistence.size=15Gi --set persistence.accessModes={"ReadWriteMany"} --set postgresql.livenessProbe.initialDelaySeconds=300 --set postgresql.readinessProbe.initialDelaySeconds=240
```

## Connection

Get the password for the postgres (admin) user

```bash
kubectl get secret -n postgresql postgresql-postgresql-ha-postgresql -o jsonpath='{.data.\password}' | base64 -d
```

Access the postgresql-postgresql-ha-pgpool service on port 5432 (eventually through kubectl proxy) with user postgres and just found password

```bash
kubectl port-forward -n postgresql service/postgresql-postgresql-ha-pgpool :5432
```

You can also use the service dns inside the cluster: `postgresql-postgresql-ha-pgpool.postgresql.svc.cluster.local`

## Add users to pgpool



```bash
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 -d
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-pgpool -o jsonpath="{.data.admin-password}" | base64 -d
helm upgrade -i postgresql bitnami/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set persistence.storageClass=longhorn-nvme --set persistence.size=15Gi --set persistence.accessModes={"ReadWriteMany"} --set pgpool.customUsers.usernames="user01,user02" --set pgpool.customUsers.passwords="pwd01,pwd02" --set postgresql.password=<postres password> --set postgresql.repmgrPassword=<repmgr-password> --set pgpool.adminPassword=<pgpool admin password>
```

## Pgadmin

Change the password in the secret, set the postgresql server(s) info in the configmap, change the domains in the certificate and ingressroutes, then apply the yaml.

```bash
kubectl apply -n pgadmin.yaml
```

You'll probably need to go through the password resetting tool, then it will ask for the postgres user password (select to save it)
