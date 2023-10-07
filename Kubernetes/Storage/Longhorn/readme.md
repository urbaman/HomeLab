# Longhorn kubernetes storage solution

## Environment check

Download and run the environment check

```bash
sudo apt install jq nfs-common -y
curl -sSfL https://raw.githubusercontent.com/longhorn/longhorn/v1.5.1/scripts/environment_check.sh | bash
```

## Installation through manifests

```bash
wget https://raw.githubusercontent.com/longhorn/longhorn/v1.5.1/deploy/longhorn.yaml
```

Edit the settings adding needed lines to the longhorn-default-setting ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-default-setting
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.5.1
data:
  default-setting.yaml: |-
    default-data-path: /longhorn/vdisk/ # this is the path where the primary storage is mounted
    storage-network: default/ipvlan-conf # this is the name of the multus network-attachment-definition
    default-replica-count: 3
```

Eventually set the number of replicas in the StorageClass, and see the data-locality (`best-effort tries` to put a replica on the same node where the workload runs, it can be `disabled` or `strict-local` for a single replica) and the node/disk selectors (if you need to deploy different layers of storage, see below) then install.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: longhorn-storageclass
  namespace: longhorn-system
  labels:
    app.kubernetes.io/name: longhorn
    app.kubernetes.io/instance: longhorn
    app.kubernetes.io/version: v1.5.1
data:
  storageclass.yaml: |
    kind: StorageClass
    apiVersion: storage.k8s.io/v1
    metadata:
      name: longhorn-vdisk
      annotations:
        storageclass.kubernetes.io/is-default-class: "true"
    provisioner: driver.longhorn.io
    allowVolumeExpansion: true
    reclaimPolicy: "Delete"
    volumeBindingMode: Immediate
    parameters:
      numberOfReplicas: "3"
      staleReplicaTimeout: "30"
      fromBackup: ""
      fsType: "ext4"
      dataLocality: "best-effort"
      diskSelector: "vdisk"
      nodeSelector: "storage"

```


```bash
kubectl apply -f longhorn.yaml
```

## Installation through helm (can't manage some of the settings)

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm upgrade -i longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.5.1 --set defaultSettings.defaultDataPath=/longhorn/storage/ --set defaultSettngs.storageNetwork=default/ipvlan-conf --set defaultSettings.defaultReplicaCount=3 --set persistence.defaultClassReplicaCount=3 
```

Or, get the values file, change the settings and install

```bash
helm show values longhorn/longhorn > longhorn-values.yaml
helm upgrade -i longhorn longhorn/longhorn --namespace longhorn-system --create-namespace --version 1.5.1 --values longhorn-values.yaml
```

## Access the gui

From your client with kubectl installed:

```bash
kubectl port-forward -n longhorn-system service/longhorn-frontend :80
```

Go to <http://IP-OF-A-K8S-CONTROL-PANEL:PORT-SHOWN-IN-OUTPUT>

Check that all nodes are running and scheduling, and that the storage space is as expected.

## Check

### With PV

Create a volume from the Longhorn UI named test-volume, then create the PV (beware to use the volume name in volumeHandle), PVC and POD

```yaml
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

```yaml
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

## Different Storage layers

If you want to manage different storage layers:

- Add disk(s) to two separate paths on host(s) (`/longhorn/vdisk`, `/longhorn/nvme`), deploy longhorn with the default path (`/longhorn/vdisk`) nodeSeleceotr and diskSelector enabled in the default storageClass, defining the storage layer used as default
- Go to the GUI, add the second disk (path) to the nodes, tag the nodes as `default` and the two disks as `vdisk` or `nvme`
- Deploy a second storageClass pointing the diskSelector to the second storage layer (disks), see example in the yamls dir
- Try the above dynamic pvc/pod example doubling up with a version using the new storage class

## Exposing the dashboard with Traefik

Apply the ig-longhorn.yaml template.
