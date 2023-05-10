# Longhorn kubernetes storage solution


## Installation


```
wget https://raw.githubusercontent.com/longhorn/longhorn/v1.4.1/deploy/longhorn.yaml
```

Edit the settings adding needed lines to the longhorn-default-setting ConfigMap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-default-setting
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.4.1
data:
  default-setting.yaml: |-
    default-data-path: /longhorn/storage/
    storage-network: default/ipvlan-conf
    default-replica-count: 6
```

Eventually set the nomber of replicas in the StorageClass, then install.

```
kubectl apply -f longhorn.yaml
```

## Access the gui

From your client with kubectl installed:

```
kubectl port-forward -n longhiorn-system service/longhorn-frontend :80
```

Go to http://IP-OF-A-K8S-CONTROL-PANEL:PORT-SHOWN-IN-OUTPUT

Check that all nodes are running and scheduling, and that the storage space is as expected.

## Check

### With PV

Create a volume from the Longhorn UI named test-volume, then create the PV (beware to use the volume name in volumeHandle), PVC and POD

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: longhorn-vol-pv
spec:
  capacity:
    storage: 2Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: longhorn
  csi:
    driver: driver.longhorn.io
    fsType: ext4
    volumeAttributes:
      numberOfReplicas: '3'
      staleReplicaTimeout: '2880'
    volumeHandle: test-volume
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-vol-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
  volumeName: longhorn-vol-pv
  storageClassName: longhorn
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-pv-test
  namespace: default
spec:
  restartPolicy: Always
  containers:
  - name: volume-pv-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
          - ls
          - /data/lost+found
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: vol
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: vol
    persistentVolumeClaim:
      claimName: longhorn-vol-pvc
```

### Dynamic PVC

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-volv-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
  namespace: default
spec:
  restartPolicy: Always
  containers:
  - name: volume-test
    image: nginx:stable-alpine
    imagePullPolicy: IfNotPresent
    livenessProbe:
      exec:
        command:
          - ls
          - /data/lost+found
      initialDelaySeconds: 5
      periodSeconds: 5
    volumeMounts:
    - name: volv
      mountPath: /data
    ports:
    - containerPort: 80
  volumes:
  - name: volv
    persistentVolumeClaim:
      claimName: longhorn-volv-pvc
```

## Prometheus

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: longhorn-prometheus-servicemonitor
  namespace: monitoring
  labels:
    name: longhorn-prometheus-servicemonitor
    release: kube-prometheus-stack
spec:
  selector:
    matchLabels:
      app: longhorn-manager
  namespaceSelector:
    matchNames:
    - longhorn-system
  endpoints:
  - port: manager
```

Import a suitable grafana dashboard (Longhorn Example is ok), get the json definition and create the json definition file.

```
kubectl create configmap grafana-dashboard-longhorn --from-file=grafana-longhorn.json
kubectl label configmap grafana-dashboard-longhorn grafana_dashboard="1"
```
