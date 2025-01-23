# Postgresql

## Installation

We need to give time for the pods to be ready (due to shared storage latency) in liveness and readiness probes, and set a high number of concurrent connections for Gitlab in `pgpool.maxPool`, `pgpool.numInitChildren` and `postgresql.maxConnections`. Anyway, `pgpool.maxPool`*`pgpool.numInitChildren`=`postgresql.maxConnections`/2, and must be enough for connection-spawning applications.
Also set higher resource limits and requests (double memory values).

```bash
#helm repo add bitnami https://charts.bitnami.com/bitnami
#helm repo update
helm show values oci://registry-1.docker.io/bitnamicharts/postgresql > postgresql-values.yaml
```

Change Max Connections, Persistence, Metrics, and check compatible version for apps (Gitlab for example, version 14.x at the moment):

```yaml
image:
  teg: 14.15.0-debian-12-r8
primary:
  extraEnvVars:
    - name: POSTGRESQL_MAX_CONNECTIONS
      value: "1024"
  persistence:
    storageClass: "asustornas1-nfs"
    size: 15Gi
  resourcePreset: "medium"
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    labels:
      release: kube-prometheus-stack
```

Then install

```bash
helm upgrade -i postgresql oci://registry-1.docker.io/bitnamicharts/postgresql --namespace postgresql --create-namespace -f postgresql-values.yaml
```

## Connection

Get the password for the postgres (admin) user

```bash
kubectl get secret -n postgresql postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode
```

Access the postgresql service on port 5432 (eventually through kubectl proxy) with user postgres and just found password

```bash
kubectl port-forward -n postgresql service/postgresql :5432
```

You can also use the service dns inside the cluster: `postgresql.postgresql.svc.cluster.local`

You can use a postgresql pod to connect and check the db and connections:

```bash
kubectl exec -it -n postgresql <POSTGRESQL_POD> -- psql -d postgres -U postgres -h postgresql.postgresql.svc.cluster.local
```

## Pgadmin

Change the password in the secret, set the pgadmin_default_email, the postgresql server(s) info in the configmap, change the domains in the certificate and ingressroutes, then apply the yaml.

```bash
kubectl apply -n pgadmin.yaml
```

You'll probably need to go through the password resetting tool, then it will ask for the postgres user password (select to save it)

## Expose through Traefik

Define an entrypoint for postgresql (port 5432) in Traefik, then deploy the `ig-postgresql.yaml` file

## Upgrade to mayor version

```bash
kubectl exec -it -n postgresql postgresql-0 -- bash
postgres --version
```

Set the same version in <pg-version>, set the new image version in <pg-new-image>

- Deploy the new db with the following flags (and the same root password: `--set auth.postgresPassword=$POSTGRESQL_PASSWORD`):

```yaml
primary:
  containerSecurityContext:
    runAsUser: 0
    runAsNonRoot: false
diagnosticMode:
  enabled: true
```

- Update the old db with the following flags:

```yaml
primary:
  containerSecurityContext:
    runAsUser: 0
    runAsNonRoot: false
diagnosticMode:
  enabled: true
```

- Stop the old db:

```bash
kubectl exec -it -n <namespace> <old_pod> -- bash
pg_ctl stop
exit
```

- Copy `/opt/bitnami/bin` in `/opt/bitnami/oldbin` and `/bitnami/data` in `/bitnami/olddata` from old to new

```bash
mkdir /path #create tem path
mkdir /path/oldbin #create temp path for bin
kubectl cp <namespace><old_pod>:/opt/bitnami/bin /path
kubectl cp <namespace><old_pod>:/bitnami/postgresql/bin /path/oldbin
mv /path/data /path/olddata
kubectl cp /path <namespace><new_pod>:/bitnami
```

- eventually, create the following symlinks in the new pod, and give the right permissions to the olddata directory

```bash
kubectl exec -it -n <namespace> <new_pod> -- bash
ln -sf /bitnami/oldbin/geod /bitnami/oldbin/invgeod
ln -sf /bitnami/oldbin/proj /bitnami/oldbin/invproj
ln -sf /bitnami/oldbin/postgres /bitnami/oldbin/postmaster
chown -R postgres:postgres /bitnami/olddata
```

- Start the upgrade

```bash
pg_upgrade -c -b /bitnami/oldbin -B /opt/bitnami/postgresql/bin -d /bitnami/olddata -D /bitnami/postgresql/data
exit
```

- Update the new db with:

```yaml
primary:
  containerSecurityContext:
    runAsUser: 1001
    runAsNonRoot: true
diagnosticMode:
  enabled: false
```

- Verify the connection to new db
- Eventually, copy the old svc, delete the old chart and recreate the svc pointing to the new one so all old connections can work
