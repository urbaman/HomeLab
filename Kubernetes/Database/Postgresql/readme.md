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
primary:
  extraEnvVars:
    - name: POSTGRESQL_MAX_CONNECTIONS
      value: 1024
  persistence:
    storageClass: "asustornas1-nfs"
    size: 15Gi
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

## Upgrade to mayor version:

```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)
helm upgrade postgresql bitnami/postgresql \
  --set image.tag=12.8.0-debian-10-r90 \ #new image
  --set containerSecurityContext.runAsUser=0 \
  --set diagnosticMode.enabled=true \
  --set postgresqlPassword=$POSTGRESQL_PASSWORD
kubectl exec -it -n postgresql postgresql-0 -- bash
postgres --version
. /opt/bitnami/scripts/libos.sh
ensure_group_exists postgres -i 1001
ensure_user_exists postgres -i 1001 -g postgres
#root@postgresql-postgresql-0:/tmp# mv /bitnami/postgresql/data /bitnami/postgresql/olddata
#root@postgresql-postgresql-0:/tmp# mkdir -p /bitnami/postgresql/data mkdir -p /bitnami/postgresql/oldbin
#root@postgresql-postgresql-0:/tmp# chown -R postgres:postgres /bitnami/postgresql/data /bitnami/postgresql/olddata
#root@postgresql-postgresql-0:/# cd /tmp/
#root@postgresql-postgresql-0:/# curl --remote-name --silent https://downloads.bitnami.com/files/stacksmith/postgresql-11.13.0-14-linux-amd64-debian-10.tar.gz
#root@postgresql-postgresql-0:/# tar --extract --directory /bitnami/postgresql/oldbin/ --file postgresql-11.13.0-14-linux-amd64-debian-10.tar.gz --strip-components=4 postgresql-11.13.0-14-linux-amd64-debian-10/files/postgresql/bin
#root@postgresql-postgresql-0:/tmp# gosu postgres initdb -E UTF8 -D /bitnami/postgresql/data -U postgres
#(...)
#root@postgresql-postgresql-0:/tmp# gosu postgres pg_upgrade -c -b /bitnami/postgresql/oldbin -B /opt/bitnami/postgresql/bin -d /bitnami/postgresql/olddata -D /bitnami/postgresql/data
#(...)
pg_upgrade
```
