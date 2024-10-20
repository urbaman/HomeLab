# Postgresql

## Installation

We need to give time for the pods to be ready (due to shared storage latency) in liveness and readiness probes, and set a high number of concurrent connections for Gitlab in `pgpool.maxPool`, `pgpool.numInitChildren` and `postgresql.maxConnections`. Anyway, `pgpool.maxPool`*`pgpool.numInitChildren`=`postgresql.maxConnections`/2, and must be enough for connection-spawning applications.

```bash
#helm repo add bitnami https://charts.bitnami.com/bitnami
#helm repo update
helm show values postgresql oci://registry-1.docker.io/bitnamicharts/postgresql > postgresql-values.yaml
```

Change Max Connections, Persistence, Metrics:

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
