# Mysql db

## Installation

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mysql bitnami/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=longhorn --set primary.persistence.size=15Gi --set primary.persistence.accessModes={ReadWriteMany} --set secondary.persistence.storageClass=longhorn --set secondary.persistence.size=15Gi --set secondary.persistence.accessModes={ReadWriteMany} --set architecture=replication --set secondary.replicaCount=2 --set primary.livenessProbe.initialDelaySeconds=480 --set primary.readinessProbe.initialDelaySeconds=120 --set secondary.livenessProbe.initialDelaySeconds=480 --set secondary.readinessProbe.initialDelaySeconds=120
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
