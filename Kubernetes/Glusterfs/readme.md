# Storage (GlusterFS)

## Having a cluster running

Follow a guide, mine's [here](https://github.com/urbaman/HomeLab/tree/main/Storage/Glusterfs)

## glusterfs-client

Fist you need to install the glusterfs-client package on your nodes. The client is used by the kubernetes scheduler to create the gluster volumes.

```bash
$ sudo apt install gluster-client
```

## Discovering GlusterFS in Kubernetes:

GlusterFS cluster should be discovered in the Kubernetes cluster. To do that, you need to add an Endpoints object that points to the servers of the GlusterFS cluster.

```bash
apiVersion: v1
kind: Endpoints
metadata:
  name: glusterfs-cluster
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
```

To persist the Gluster endpoints, you also need to create a relative service.

```bash
apiVersion: v1
kind: Service
metadata:
  name: glusterfs-cluster 
spec:
  ports:
  - port: 1
```

## Using GlusterFS in Kubernetes:

Create the desired path in the glusterfs volume (HDD5T/path/... in my situation)

### Method 1 — Connecting to GlusterFS directly with Pod manifest:

To connect to the GlusterFS volume directly with Pod manifest, use the GlusterfsVolumeSource in the Pod. Here is an example (create the path in the glusterfs volume - HDD5T in my situation):

```bash
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
    - name: alpine
      image: alpine:latest
      command:
        - touch
        - /data/test
      volumeMounts:
        - name: glusterfs-volume
          mountPath: /data
  volumes:
    - name: glusterfs-volume
      glusterfs:
        endpoints: glusterfs-cluster
        path: HDD5T/path/...
        readOnly: no
```

### Method 2 — Connecting using the PersistentVolume resource:

To create the PersistentVolume object for the GlusterFS volume, use the following manifest. The storage size does not take any effect. Be sure to name the claim you'll bind it to.

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-path
  namespace: project-namespace
spec:
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Gi
  storageClassName: ""
  volumeMode: Filesystem
  glusterfs:
    endpoints: glusterfs-cluster
    path: HDD5T/path/...
    readOnly: false
  claimRef:
    name: glusterfs-path-pvc
    namespace: project-namespace
  persistentVolumeReclaimPolicy: Retain
```

Then create the relative PersistentVolumeClaim (be sure to name the PersistentVolume to Claim):

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: glusterfs-path-pvc
    namespace: project-namespace
spec:
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 10Gi
    storageClassName: ""
    volumeName: glusterfs-path
```

And then, use the claim as a volume in the pods:

```bash
apiVersion: v1
kind: Pod
metadata:
    name: mypod
    namespace: project-namespace
spec:
    containers:
      - name: myfrontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: mypd
    volumes:
      - name: mypd
        persistentVolumeClaim:
          claimName: glusterfs-path-pvc
```
