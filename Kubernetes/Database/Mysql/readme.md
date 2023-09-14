# Mysql db

## Installation - Replica cluster

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql bitnami/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=longhorn --set primary.persistence.size=15Gi --set primary.persistence.accessModes={ReadWriteMany} --set secondary.persistence.storageClass=longhorn --set secondary.persistence.size=15Gi --set secondary.persistence.accessModes={ReadWriteMany} --set architecture=replication --set secondary.replicaCount=2 --set primary.livenessProbe.initialDelaySeconds=600 --set primary.readinessProbe.initialDelaySeconds=600 --set secondary.livenessProbe.initialDelaySeconds=600 --set secondary.readinessProbe.initialDelaySeconds=600 --set primary.startupProbe.initialDelaySeconds=600 --set secondary.startupProbe.initialDelaySeconds=600
```

## Installation - standalone

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql bitnami/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=longhorn --set primary.persistence.size=15Gi --set primary.persistence.accessModes={ReadWriteMany} --set primary.livenessProbe.initialDelaySeconds=480 --set primary.readinessProbe.initialDelaySeconds=480 --set primary.startupProbe.initialDelaySeconds=480
```

## Connection

Get the password for the root (admin) user

```bash
kubectl get secret -n mysql mysql -o jsonpath='{.data.mysql-root-password}' | base64 -d
```

Access the mysql service on port 5432 (eventually through kubectl proxy) with user root and just found password

```bash
kubectl port-forward -n mysql service/mysql-primary :3306
```

You can also use the service dns inside the cluster: mysql.mysql-primary.svc.cluster.local

## Proxysql for read/write split

Add a monitoring user ro mysql:

```sql
CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
GRANT USAGE, REPLICATION CLIENT ON *.* TO 'monitor'@'%';
```

Rename the proxysql-mysql.cnf file to proxysql.cnf

```bash
kubectl create configmap -n mysql proxysql-configmap --from-file=proxysql.cnf
kubectl apply -f proxysql-mysql.yaml
```

Now connect through the proxysql service or proxysql.mysql.svc.cluster.local

## Phpmyadmin for management

```bash
kubectl apply -f phpmyadmin-mysql.sql
```
