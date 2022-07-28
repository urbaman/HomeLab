# Nextcloud

## What we need

To properly install nextcloud, we will need the following:

- a DNS entry, reachable (see cloudflare operator)
- a MySQL database (we will use mariadb)
- a Redis instance

We will install all of the components, together with their prometheus-grafana monitoring.

After having set the proper DNS record (we consider it done), we start our process.

### Persistent volumes

First, we will need persistent volumes for all of the apps:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
  namespace: nextcloud
subsets:
  - addresses:
      - ip: 10.0.50.21
        hostname: truenas1
      - ip: 10.0.50.22
        hostname: truenas2
      - ip: 10.0.50.23
        hostname: truenas3
    ports:
      - port: 1
---
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster
  namespace: nextcloud
spec:
  ports:
  - port: 1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nextcloud-mariadb-pv
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 8Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/nextcloud/mariadb
    readOnly: false
  claimRef:
    name: nextcloud-mariadb-pvc
    namespace: nextcloud
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: nextcloud-mariadb-pvc
    namespace: nextcloud
spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 8Gi
    storageClassName: ""
    volumeName: nextcloud-mariadb-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nextcloud-pv
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 8Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/nextcloud/nextcloud
    readOnly: false
  claimRef:
    name: nextcloud-pvc
    namespace: nextcloud
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: nextcloud-pvc
    namespace: nextcloud
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 8Gi
    storageClassName: ""
    volumeName: nextcloud-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nextcloud-data-pv
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 5Ti
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/nextcloud/data
    readOnly: false
  claimRef:
    name: nextcloud-data-pvc
    namespace: nextcloud
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: nextcloud-data-pvc
    namespace: nextcloud
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 5Ti
    storageClassName: ""
    volumeName: nextcloud-data-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-master-pv
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 8Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/nextcloud/redis/master
    readOnly: false
  claimRef:
    name: redis-master-pvc
    namespace: nextcloud
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: redis-master-pvc
    namespace: nextcloud
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 8Gi
    storageClassName: ""
    volumeName: redis-master-pv
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-replica-pv
  namespace: nextcloud
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 8Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/k8storage/nextcloud/redis/replica
    readOnly: false
  claimRef:
    name: redis-replica-pvc
    namespace: nextcloud
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: redis-replica-pvc
    namespace: nextcloud
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 8Gi
    storageClassName: ""
    volumeName: redis-replica-pv
```

### Namespace

We will use the `nextcloud` namespace for our instances.

```bash
kubectl create namespace nextcloud
```

### Secrets

After the PVCs, we need some secrets (passwords and so on).

We start by getting the base64 version of our strings:

```bash
echo 'dbname' | base64
echo 'password' | base64
echo 'smtp-password' | base64
```

And then we actually create the secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nextcloud-secrets
  namespace: nextcloud
type: Opaque
data:
  MYSQL_DATABASE: dbname-base64
  MYSQL_USER: dbname-base64
  MYSQL_PASSWORD: password-base64 
  MYSQL_ROOT_PASSWORD: password-base64
  REDIS_HOST_PASSWORD: password-base64
  SMTP_PASSWORD: smtp-password-base64
```

### MariaDB

With the PVCs and the secret done, we can go on with mariadb. In the yaml file below, we already added all of the components (deployment, service, exporter and service monitor) for the instance and its monitoring.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mariadb
  namespace: nextcloud 
  labels:
    app: mariadb 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mariadb 
  template:
    metadata:
      labels:
        app: mariadb 
    spec:
      volumes:
      - name: mariadb-data
        persistentVolumeClaim:
          claimName: nextcloud-mariadb-pvc
      containers:
      - name: mariadb 
        image: mariadb:latest 
        ports:
          - containerPort: 3306 
        args:
          - --transaction-isolation=READ-COMMITTED
          - --binlog-format=ROW
          - --max-connections=1000
        env:
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  key: MYSQL_DATABASE
                  name: nextcloud-secrets
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: MYSQL_PASSWORD
                  name: nextcloud-secrets
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  key: MYSQL_USER
                  name: nextcloud-secrets
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: MYSQL_ROOT_PASSWORD
                  name: nextcloud-secrets
        volumeMounts:
        - name: mariadb-data
          mountPath: /var/lib/mysql
---
apiVersion: v1
kind: Service
metadata:
  name: mariadb
  namespace: nextcloud
  labels:
    app: mariadb
spec:
  ports:
    - port: 3306
  selector:
    app: mariadb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysqld-exporter
  namespace: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysqld-exporter
  template:
    metadata:
      labels:
        app: mysqld-exporter
    spec:
      containers:
      - name: mysqld-exporter
        image: prom/mysqld-exporter:main
        args:
        - --collect.info_schema.tables
        - --collect.info_schema.innodb_metrics
        - --collect.global_status
        - --collect.global_variables
        - --collect.info_schema.processlist
        - --collect.perf_schema.tablelocks
        - --collect.perf_schema.eventsstatements
        - --collect.perf_schema.eventswaits
        - --collect.auto_increment.columns
        - --collect.binlog_size
        - --collect.perf_schema.tableiowaits
        - --collect.perf_schema.indexiowaits
        - --collect.info_schema.userstats
        - --collect.info_schema.clientstats
        - --collect.info_schema.tablestats
        - --collect.info_schema.schemastats
        - --collect.perf_schema.file_events
        - --collect.perf_schema.file_instances
        - --collect.info_schema.innodb_cmp
        - --collect.info_schema.innodb_cmpmem
        - --collect.info_schema.query_response_time
        - --collect.engine_innodb_status
        ports:
        - containerPort: 9104
          protocol: TCP
        env:
        - name: DATA_SOURCE_NAME
          value: "mysqld-exporter:dryMfRVXgT4zoyI9@(mariadb.nextcloud.svc.cluster.local:3306)/"
---
apiVersion: v1
kind: Service
metadata:
  name: mysqld-exporter
  namespace: nextcloud
  labels:
    app: mysqld-exporter
spec:
  type: ClusterIP
  ports:
  - port: 9104
    protocol: TCP
    name: http
  selector:
    app: mysqld-exporter
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mysqld-exporter
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  jobLabel: nextcloud-mariadb
  endpoints:
    - interval: 5s
      port: http
  namespaceSelector:
    matchNames:
    - nextcloud
  selector:
    matchLabels:
      app: mysqld-exporter
```

Now we only need to add the grafana dashboard and label it.

```bash
kubectl create configmap grafana-dashboard-mariadb -n monitoring --from-file=04-nextcloud-mariadb-grafana.json
kubectl label configmap grafana-dashboard-mariadb -n monitoring grafana_dashboard="1"
```

### Redis

Same process goes for Redis, in a Master and 3 Replicas setting, but we use helm this time, with our yaml as values.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install -n nextcloud redis bitnami/redis --values 05-nextcloud-redis.yaml 
```

Now we only need to add the grafana dashboard and label it.

```bash
kubectl create configmap grafana-dashboard-redis -n monitoring --from-file=06-nextcloud-redis-grafana.json
kubectl label configmap grafana-dashboard-redis -n monitoring grafana_dashboard="1"
```

### Nextcloud deployment

If we correctly deployed PVCs, secrets, mariadb and redis, we can go on with nextcloud, adding the secrets and some other useful settings. We also added the Traefik Ingress Route, with https and ssl certs.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud
  namespace: nextcloud 
  labels:
    app: nextcloud 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud 
  template:
    metadata:
      labels:
        app: nextcloud
    spec:
      volumes:
      - name: nextcloud-app
        persistentVolumeClaim: 
          claimName: nextcloud-pvc
      - name: nextcloud-data
        persistentVolumeClaim: 
          claimName: nextcloud-data-pvc
      containers:
        - image: nextcloud:apache
          name: nextcloud 
          ports:
            - containerPort: 80
          env:
            - name: REDIS_HOST
              value: redis-master
            - name: MYSQL_HOST
              value: mariadb 
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  key: MYSQL_DATABASE
                  name: nextcloud-secrets
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: MYSQL_PASSWORD
                  name: nextcloud-secrets
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  key: MYSQL_USER
                  name: nextcloud-secrets
            - name: NEXTCLOUD_ADMIN_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: MYSQL_PASSWORD
                  name: nextcloud-secrets
            - name: REDIS_HOST_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: REDIS_HOST_PASSWORD
                  name: nextcloud-secrets
            - name: NEXTCLOUD_ADMIN_USER
              value: "admin"
            - name: SMTP_HOST
              value: "smtp.urbaman.it"
            - name: SMTP_SECURE
              value: "tls"
            - name: SMTP_PORT
              value: "587"
            - name: SMTP_NAME
              value: "nextcloud@urbaman.it"
            - name: MAIL_FROM_ADDRESS
              value: "nextcloud@urbaman.it"
            - name: NEXTCLOUD_TRUSTED_DOMAINS
              value: "urbaman.it"
            - name: TRUSTED_PROXIES
              value: "10.0.50.100"
            - name: OVERWRITEHOST
              value: "nextcloud.urbaman.it"
            - name: OVERWRITEPROTOCOL
              value: "https"
            - name: OVERWRITECLIURL
              value: "nextcloud.urbaman.it"
            - name: SMTP_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: SMTP_PASSWORD
                  name: nextcloud-secrets 
          volumeMounts:
            - mountPath: /var/www/html
              name: nextcloud-app
            - mountPath: /var/www/html/data
              name: nextcloud-data
---
apiVersion: v1
kind: Service
metadata:
  name: nextcloud
  namespace: nextcloud
  labels:
    app: nextcloud
spec:
  ports:
    - port: 8080
      targetPort: 80
  selector:
    app: nextcloud
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: nextcloud-https-redirect
  namespace: nextcloud
spec:
  redirectScheme:
    scheme: https
    permanent: true
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nextcloud-websecure
  namespace: nextcloud
spec:
  entryPoints:
    - websecure
  routes:
  - match: Host(`nextcloud.urbaman.it`)
    kind: Rule
    services:
    - name: nextcloud
      port: 8080
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: nextcloud-web
  namespace: nextcloud
spec:
  entryPoints:
    - web
  routes:
  - match: Host(`nextcloud.urbaman.it`)
    kind: Rule
    services:
    - name: nextcloud
      port: 8080
    middlewares:
      - name: nextcloud-https-redirect
```

### Nextcloud Monitoring

Finally, we deploy the nextcloud exporter for monitoring, but we need another step before that: creating an auth token for nextcloud, using its occ function.

#### Create an Auth Token

```bash
kubectl exec -it -n nextcloud <nextcloud_pod_name> -- bash
TOKEN=$(openssl rand -hex 32)
su -s /bin/bash www-data -c "php occ config:app:set serverinfo token --value "$TOKEN""
```

#### Monitoring Deploy

And then, the deploy, changing the auth-token with the one just generated.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextcloud-exporter
  namespace: nextcloud
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nextcloud-exporter
  template:
    metadata:
      labels:
        app: nextcloud-exporter
    spec:
      containers:
      - name: nextcloud-exporter
        image: xperimental/nextcloud-exporter:latest
        ports:
        - containerPort: 9205
          name: metrics
        env:
        - name: NEXTCLOUD_AUTH_TOKEN
          value: "auth_token"
        - name: NEXTCLOUD_SERVER
          value: "https://nextcloud.urbaman.it"
        - name: NEXTCLOUD_TIMEOUT
          value: "5s"
---
apiVersion: v1
kind: Service
metadata:
  name: nextcloud-exporter
  namespace: nextcloud
  labels:
    app: nextcloud-exporter
spec:
  type: ClusterIP
  ports:
  - port: 9205
    protocol: TCP
    name: http
  selector:
    app: nextcloud-exporter
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nextcloud-exporter
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  jobLabel: nextcloud
  endpoints:
    - interval: 5s
      port: http
  namespaceSelector:
    matchNames:
    - nextcloud
  selector:
    matchLabels:
      app: nextcloud-exporter
```

Last step: the grafana dashboard.

```bash
kubectl create configmap grafana-dashboard-nextcloud -n monitoring --from-file=09-nextcloud-grafana.json
kubectl label configmap grafana-dashboard-nextcloud -n monitoring grafana_dashboard="1"
```