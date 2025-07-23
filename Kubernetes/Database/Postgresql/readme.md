# Postgresql

## Installation

### CloudnativePG

#### Operator

In Kubernetes, the operator is by default installed in the cnpg-system namespace as a Kubernetes Deployment. The name of this deployment depends on the installation method. When installed through the manifest or the cnpg plugin, it is called cnpg-controller-manager by default. When installed via Helm, the default name is cnpg-cloudnative-pg.

Install the kubectl cnpg plugin

```bash
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
```

Install via the plugin

```bash
kubectl cnpg install generate \
  --watch-namespaces "cnpg-system" \
  > cnpg_for_cnpg-system.yaml
```

Or via helm

```bash
helm repo add cnpg https://cloudnative-pg.github.io/charts
helm repo update
helm upgrade -i cnpg \
  --namespace cnpg-system \
  --create-namespace \
  cnpg/cloudnative-pg
```

#### Database (via helm)

```bash
helm show values cnpg/cluster > cnpg-cluster-values.yaml
helm upgrade --install database \
  --namespace database \
  --create-namespace \
  cnpg/cluster -f cnpg-cluster-values.yaml
```

#### Database (via CRD, for more ample customization)

```bash
kubectl apply -f cnpg-cluster.yaml
```

#### Connection

Get the superuser (postgres) password

```bash
kubectl get secret -n <namespace> <cluster-name>-superuser -o jsonpath="{.data.password}" | base64 --decode
```

And connct to <cluister-name>-rw.<namespace>(.svc.cluster.local)

### Bitnami

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
helm upgrade -i postgresql-<major-version> oci://registry-1.docker.io/bitnamicharts/postgresql --namespace postgresql --create-namespace -f postgresql-values.yaml
```

#### Connection

Get the password for the postgres (admin) user

```bash
kubectl get secret -n postgresql postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode
```

Copy the svc, and deploy it again changing the name to postgresql

```bash
kubectl get svc -n postgresql postgresql-<major-version> -o yaml > postgresql-defualt-svc.yaml
vi postgresql-defualt-svc.yaml
kubectl apply -f postgresql-defualt-svc.yaml
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

#### Upgrade to mayor version

- Deploy the new db with the same root password: `--set auth.postgresPassword=$POSTGRESQL_PASSWORD`
- Delete the old postgresql svc, and re-create it from the postgresql-<new> one
- Deploy a new db `postrgesql-client` release name, the same image as the new db and the following values:

```yaml
primary:
  containerSecurityContext:
    runAsUser: 0
    runAsNonRoot: false
    readOnlyRootFilesystem: false
  persistence:
    enable: false
  resourcesPreset: "nano"
diagnosticMode:
  enabled: true
```

- Disable the login property of all the non-postgres roles on the old db (I use pgAdmin) **NB:** If you already swapped the postgresql svc, this shouldn't be necessary
- Connect to the postgresql-client and run the upgrade

```bash
kubectl exec -it -n postgresql postgresql-client-0 -- /bin/bash
pg_dumpall -U postgres -h postgresql-<old>.postgresql.svc.cluster.local --clean --file=mydb_backup.dump
psql -h postgresql-<new>.postgresql.svc.cluster.local -U postgres -f mydb_backup.dump postgres
rm mydb_backup.dump
```

- Verify the connection to the new db, and re-enable the login property of the roles (I use pgAdmin)
- Verify all the apps using the database
- Eventually, copy the old svc, delete the old chart and recreate the svc pointing to the new one so all old connections can work

**NB:** You can also try without the client deployment, using the new db deployemnt directly:

```bash
kubectl exec -it -n postgresql postgresql-<new>-0 -- /bin/bash
pg_dumpall -U postgres -h postgresql-<old>.postgresql.svc.cluster.local --clean --file=mydb_backup.dump
psql -h postgresql-<new>.postgresql.svc.cluster.local -U postgres -f mydb_backup.dump postgres
rm mydb_backup.dump
```

Or (even better?):

```bash
kubectl exec -it -n postgresql postgresql-<new>-0 -- pg_dumpall -U postgres -h postgresql-<old>.postgresql.svc.cluster.local --clean --file=mydb_backup.dump
kubectl exec -it -n postgresql postgresql-<new>-0 -- psql -h postgresql-<new>.postgresql.svc.cluster.local -U postgres -f mydb_backup.dump postgres
kubectl exec -it -n postgresql postgresql-<new>-0 -- rm mydb_backup.dump
```

## Pgadmin

Change the password in the secret, set the pgadmin_default_email, the postgresql server(s) info in the configmap, change the domains in the certificate and ingressroutes, then apply the yaml.

```bash
kubectl apply -f pgadmin.yaml
```

You'll probably need to go through the password resetting tool, then it will ask for the postgres user password (select to save it)

## Expose through Traefik

Define an entrypoint for postgresql (port 5432) in Traefik, then set the `ig-postgresql.yaml` to the proper connection service and deploy the file
