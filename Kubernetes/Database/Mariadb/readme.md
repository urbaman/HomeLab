# Mysql db

## Installation - Replica cluster

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade -i mariadb oci://registry-1.docker.io/bitnamicharts/mariadb --namespace mariadb --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=rook-ceph-nvme2tb --set primary.persistence.size=15Gi --set secondary.persistence.storageClass=rook-ceph-nvme2tb --set secondary.persistence.size=15Gi --set architecture=replication --set secondary.replicaCount=2
```

## Installation - standalone

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade -i mariadb oci://registry-1.docker.io/bitnamicharts/mariadb --namespace mariadb --create-namespace --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set primary.persistence.storageClass=rook-ceph-nvme2tb --set primary.persistence.size=15Gi
```

## Upgrading

Get the passwords for root and replication (only for replica cluster) users:

```bash
kubectl get secret --namespace mariadb mariadb -o jsonpath="{.data.mariadb-root-password}" | base64 -d
kubectl get secret --namespace "mariadb" mariadb -o jsonpath="{.data.mariadb-replication-password}" | base64 -d
```

Deploy the same helm chart with all the same values and add `--set auth.rootPassword=<password> --set auth.replicationPassword=<password>`

## Connection

Access the mysql service on port 5432 (eventually through kubectl proxy) with user root and just found password.
Substitute primary with secondary to access the replica.

```bash
kubectl port-forward -n mariadb service/mariadb-primary :3306
```

You can also use the service dns inside the cluster: `mariadb-primary.mariadb.svc.cluster.local`

## Proxysql for read/write split

Add a monitoring user ro mysql:

```bash
kubectl exec -it -n mariadb mariadb-primary-0 -- mysql -h localhost -u root -pPASSWORD
```

```sql
CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitor';
GRANT USAGE, REPLICATION CLIENT ON *.* TO 'monitor'@'%';
```

Rename the proxysql-mariadb.cnf file to proxysql.cnf, set the root password.

```bash
kubectl create configmap -n mariadb proxysql-configmap --from-file=proxysql.cnf
kubectl apply -f proxysql-mariadb.yaml
```

Now connect on port 6033 through the proxysql service or `proxysql.mariadb.svc.cluster.local`

### Update Proxysql config on the run

If you need to change the mysql server_version (shouldn't be needed for mariadb):

- Delete the proxysql statefulset and the pv,pvc and volume, change the server_version global variable in the config map, recreate the statefulset.
- Run the following commands, running as many connections (points 2-5) as needed to update all of the pods in the cluster, with the needed mysql version.

```
1. kubectl exec -it -n mariadb proxysql-0 -- /bin/bash
2. mysql -u radmin -pradmin -h proxysql.mariadb.svc.cluster.local -P6032 --prompt 'ProxySQL Admin> '
3. update global_variables set variable_value = '8.0.35' where variable_name = 'mysql-server_version';
3b. INSERT INTO mysql_users(username,password,default_schema,default_hostgroup,active) VALUES ('user2','password2','schema',0,1);
4. LOAD MYSQL VARIABLES TO RUNTIME;
5. SAVE MYSQL VARIABLES TO DISK;
6. exit
```

Do the same to change any other Proxysql config on the run: delete and recreate the statefulset from scratch with a new configmap or connect to update all of the instances through the Proxysql Admin console.

## Phpmyadmin for management

```bash
kubectl apply -f phpmyadmin-mariadb.yaml
```

## Expose through Traefik (only one MariaDB/MySQL instance per cluster per entrypoint)

Beware: MySQL do not manage SSL and SNI, so we can only use ```HostSNI(`*`)``` with one entrypoint per db instance (if we have more than one MySQL/MariaDB instances)

Define an entrypoint for mariadb (port 6033?) in Traefik, then deploy the `ig-mariadb.yaml` file
