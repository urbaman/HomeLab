# Postgresql cluster

## Installation

We need to give time for the pods to be ready (due to shared storage latency) in liveness and readiness probes, and set a high number of concurrent connections for Gitlab in `pgpool.maxPool`, `pgpool.numInitChildren` and `postgresql.maxConnections`. Anyway, `pgpool.maxPool`*`pgpool.numInitChildren`=`postgresql.maxConnections`/2, and must be enough for connection-spawning applications.

```bash
#helm repo add bitnami https://charts.bitnami.com/bitnami
#helm repo update
helm upgrade -i postgresql oci://registry-1.docker.io/bitnamicharts/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set pgpool.replicaCount=3 --set persistence.storageClass=rook-ceph-nvme2tb --set persistence.size=15Gi --set pgpool.maxPool=4 --set postgresql.maxConnections=1024 --set pgpool.numInitChildren=64
```

Try this for chart v14 (probably the resoucePreset default of nano is the main problem)

```bash
helm upgrade -i postgresql oci://registry-1.docker.io/bitnamicharts/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set pgpool.replicaCount=3 --set persistence.storageClass=rook-ceph-nvme2tb --set persistence.size=15Gi --set pgpool.customUsers.usernames="gitlab" --set pgpool.customUsers.passwords="7yS^tU92Kf2cK3" --set postgresql.password=lptMehz0CU --set postgresql.repmgrPassword=UfL9ZeGefS --set pgpool.adminPassword=sy2xhJNcud --set pgpool.maxPool=4 --set postgresql.maxConnections=1024 --set pgpool.numInitChildren=64 --set postgresql.resourcesPreset=none --set pgpool.resourcesPreset=none --set metrics.resourcesPreset=none --set postgresql.containerSecurityContext.readOnlyRootFilesystem=false --set pgpool.containerSecurityContext.readOnlyRootFilesystem=false --set postgresql.containerSecurityContext.runAsGroup=0 --set pgpool.containerSecurityContext.runAsGroup=0 --set metrics.containerSecurityContext.runAsGroup=0 --set global.compatibility.openshift.adaptSecurityContext=disabled
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

You can use a pgpool pod to connect and check the db and connections:

```bash
kubectl exec -it -n postgresql <PGPOOL_POD> -- psql -d postgres -U postgres -h postgresql-postgresql-ha-pgpool.postgresql.svc.cluster.local
```

## Add users to pgpool

```bash
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-postgresql -o jsonpath="{.data.repmgr-password}" | base64 -d
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-pgpool -o jsonpath="{.data.admin-password}" | base64 -d
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-pgpool-custom-users -o jsonpath="{.data.usernames}" | base64 -d
kubectl get secret --namespace "postgresql" postgresql-postgresql-ha-pgpool-custom-users -o jsonpath="{.data.passwords}" | base64 -d
```

```bash
helm upgrade -i postgresql oci://registry-1.docker.io/bitnamicharts/postgresql-ha --namespace postgresql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set pgpool.replicaCount=3 --set persistence.storageClass=rook-ceph-nvme2tb --set persistence.size=15Gi --set pgpool.customUsers.usernames="user01,user02" --set pgpool.customUsers.passwords="pwd01,pwd02" --set postgresql.password=<postres password> --set postgresql.repmgrPassword=<repmgr-password> --set pgpool.adminPassword=<pgpool admin password> --set pgpool.maxPool=4 --set postgresql.maxConnections=1024 --set pgpool.numInitChildren=64
```

## Pgadmin

Change the password in the secret, set the postgresql server(s) info in the configmap, change the domains in the certificate and ingressroutes, then apply the yaml.

```bash
kubectl apply -n pgadmin.yaml
```

You'll probably need to go through the password resetting tool, then it will ask for the postgres user password (select to save it)

## Expose through Traefik

Define an entrypoint for postgresql (port 5432) in Traefik, then deploy the `ig-postgresql.yaml` file
