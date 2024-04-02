# Mysql db

## Installation - Replica cluster

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade -i mysql oci://registry-1.docker.io/bitnamicharts/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=rook-ceph-nvme2tb --set primary.persistence.size=15Gi --set secondary.persistence.storageClass=rook-ceph-nvme2tb --set secondary.persistence.size=15Gi --set architecture=replication --set secondary.replicaCount=2
```

From chart v10

```bash
helm upgrade -i mysql oci://registry-1.docker.io/bitnamicharts/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=rook-ceph-nvme2tb --set primary.persistence.size=15Gi  --set secondary.persistence.storageClass=rook-ceph-nvme2tb --set secondary.persistence.size=15Gi --set architecture=replication --set secondary.replicaCount=2 --set global.compatibility.openshift.adaptSecurityContext=disabled --set primary.containerSecurityContext.runAsGroup=0 --set metrics.containerSecurityContext.runAsGroup=0 --set secondary.containerSecurityContext.runAsGroup=0 --set primary.containerSecurityContext.readOnlyRootFilesystem=false --set secondary.containerSecurityContext.readOnlyRootFilesystem=false --set metrics.containerSecurityContext.readOnlyRootFilesystem=false --set primary.resourcesPreset=none --set metrics.resourcesPreset=none --set secondary.resourcesPreset=none
```

## Installation - standalone

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade -i mysql oci://registry-1.docker.io/bitnamicharts/mysql --namespace mysql --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=rook-ceph-nvme2tb --set primary.persistence.size=15Gi
```

## Upgrading

Get the passwords for root and replication (only for replica cluster) users:

```bash
kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d
kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-replication-password}" | base64 -d
```

Deploy the same helm chart with all the same values and add `--set auth.rootPassword=<password> --set auth.replicationPassword=<password>`

## Connection

Access the mysql service on port 5432 (eventually through kubectl proxy) with user root and just found password

```bash
kubectl port-forward -n mysql service/mysql-primary :3306
```

You can also use the service dns inside the cluster: `mysql-primary.mysql.svc.cluster.local`

## Proxysql for read/write split

Add a monitoring user ro mysql:

```bash
kubectl exec -it -n mysql mysql-primary-0 -- mysql -h localhost -u root -pPASSWORD
```

```sql
CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
GRANT USAGE, REPLICATION CLIENT ON *.* TO 'monitor'@'%';
```

Be sure to set the server_version to the real mysql server version your're running before proxysql deployment.
Rename the proxysql-mysql.cnf file to proxysql.cnf, set the root password.

```bash
kubectl create configmap -n mysql proxysql-configmap --from-file=proxysql.cnf
kubectl apply -f proxysql-mysql.yaml
```

Now connect on port 6033 through the proxysql service or `proxysql.mysql.svc.cluster.local`

### Update Proxysql config on the run

If you need to change the mysql server_version:

- Delete the proxysql statefulset and the pv,pvc and volume, change the server_version global variable in the config map, recreate the statefulset.
- Run the following commands, running as many connections (points 2-5) as needed to update all of the pods in the cluster, with the needed mysql version.

```bash
1. kubectl exec -it -n mysql proxysql-0 -- /bin/bash
2. mysql -u radmin -pradmin -h proxysql.mysql.svc.cluster.local -P6032 --prompt 'ProxySQL Admin> '
3. update global_variables set variable_value = '8.0.34' where variable_name = 'mysql-server_version';
4. LOAD MYSQL VARIABLES TO RUNTIME;
5. SAVE MYSQL VARIABLES TO DISK;
6. exit
```

Do the same to change any other Proxysql config on the run: delete and recreate the statefulset from scratch with a new configmap or connect to update all of the instances through the Proxysql Admin console.

## Phpmyadmin for management

```bash
kubectl apply -f phpmyadmin-mysql.yaml
```

## Expose through Traefik (only one MariaDB/MySQL instance per cluster per entrypoint)

Beware: MySQL do not manage SSL and SNI, so we can only use ```HostSNI(`*`)``` with one entrypoint per db instance (if we have more than one MySQL/MariaDB instances)

Define an entrypoint for mariadb (port 3306?) in Traefik, then deploy the `ig-mysql.yaml` file
