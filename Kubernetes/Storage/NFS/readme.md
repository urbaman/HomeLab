# Install an NFS provider

Install the NFS csi driver, as it's the now de facto k8s project for NFS.

## Preparation

You need a [running NFS server](https://github.com/urbaman/HomeLab/tree/main/Storage/NFS%20Cluster), nfs-common package installed and a `showmount` verification on all nodes.

```bash
sudo apt install nfs-common -y
showmount -e nfs.domain.com
```

Output:

```bash
Export list for nfs.urbaman.it:
/HDD5T/nfsshare/exports/HDD5T clientips/24
/HDD5T/nfsshare/exports       clientips/24
/HDD5T/nfsshare/exports/SDD2T clientips/24
```

## Deploy the NFS subdir provisioner for dynamic pv (deprecated)

### Limitations and pitfalls

The software is subject to some limitations, listed below:

- The storage space provided is not guaranteed: you can allocate more than the total shared size via NFS, because there is no check for it.
- The provisioned storage limit is not enforced. The application can expand to use all the available storage regardless of the provisioned size.
- Storage resize/expansion operations are not presently supported in any form. You will end up in an error state: Ignoring the PVC: didn't find a plugin capable of expanding the volume; waiting for an external controller to process this PVC.
- The read-write permissions are not governed by the access-mode parameter, but by the settings used in the NFS configuration

### Installation

We can install a provisioner for every share we want to use.

```bash
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
helm repo update
helm upgrade -i -n nfs-provisioning --create-namespace first-nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=y.y.y.y \
    --set nfs.path=/other/exported/path \
    --set storageClass.name=first-nfs-client \
    --set storageClass.provisionerName=k8s-sigs.io/first-nfs-subdir-external-provisioner
helm upgrade -i -n nfs-provisioning --create-namespace second-nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=y.y.y.y \
    --set nfs.path=/other/exported/path \
    --set storageClass.name=second-nfs-client \
    --set storageClass.provisionerName=k8s-sigs.io/second-nfs-subdir-external-provisioner
```

If you do not need the archive function, add `--set storageClass.archiveOnDelete=false` to the parameters (remembering the `\` to add a line)

Verify the storage classes (will not be default if you already have a default sc)

```bash
kubectl get sc
NAME                 PROVISIONER                                         RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
hdd5t-nfs-client     k8s-sigs.io/hdd5t-nfs-subdir-external-provisioner   Delete          Immediate           true                   3m50s
longhorn (default)   driver.longhorn.io                                  Delete          Immediate           true                   85d
sdd2t-nfs-client     k8s-sigs.io/sdd2t-nfs-subdir-external-provisioner   Delete          Immediate           true                   3m46s
```

## Deploy NFS Csi Driver

Kubeadm:

```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
helm upgrade -i csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace csi-driver-nfs --create-namespace --set externalSnapshotter.enabled=true
```

Microk8s:

```bash
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm repo update
helm upgrade -i csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace csi-driver-nfs --create-namespace --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet --set externalSnapshotter.enabled=true
```

Add a StorageClass and a VolumeSnapshotClass (change the details)

```bash
kubectl apply -f sc-csi-nfs.yaml
kubectl apply -f snapshotclass-csi-nfs.yaml
```

## Verify

Create a test pvc and pod for each storage class

```bash
vi nfs-test.yaml
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-claim-hdd5t
spec:
  storageClassName: hdd5t-nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
---
kind: Pod
apiVersion: v1
metadata:
  name: test-pod-hdd5t
spec:
  containers:
  - name: test-pod-hdd5t
    image: busybox:latest
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && sleep 600"
    volumeMounts:
      - name: nfs-pvc-hdd5t
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc-hdd5t
      persistentVolumeClaim:
        claimName: demo-claim-hdd5t
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-claim-sdd2t
spec:
  storageClassName: sdd2t-nfs-client
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
---
kind: Pod
apiVersion: v1
metadata:
  name: test-pod-sdd2t
spec:
  containers:
  - name: test-pod-sdd2t
    image: busybox:latest
    command:
      - "/bin/sh"
    args:
      - "-c"
      - "touch /mnt/SUCCESS && sleep 600"
    volumeMounts:
      - name: nfs-pvc-sdd2t
        mountPath: "/mnt"
  restartPolicy: "Never"
  volumes:
    - name: nfs-pvc-sdd2t
      persistentVolumeClaim:
        claimName: demo-claim-sdd2t
```

Check on the NFS server, you'll see the pvc(s) and the SUCCESS files.

```bash
ls -la /HDD5T/nfsshare/exports/HDD5T/
total 13
drwxrwxr-x 3 nobody nogroup 4096 Aug 23 16:53 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:54 default-demo-claim-hdd5t-pvc-7ce33985-1dfe-4985-a015-a534f7e03bdd
-rw-r----- 1 root   root      52 Aug 23 16:54 .rmtab
```

```bash
ls -la /HDD5T/nfsshare/exports/HDD5T/default-demo-claim-hdd5t-pvc-7ce33985-1dfe-4985-a015-a534f7e03bdd/
total 8
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:54 .
drwxrwxr-x 3 nobody nogroup 4096 Aug 23 16:53 ..
-rw-r--r-- 1 nobody nogroup    0 Aug 23 16:54 SUCCESS
```

```bash
ls -la /HDD5T/nfsshare/exports/SDD2T
total 13
drwxrwxr-x 5 nobody nogroup 4096 Aug 23 16:53 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:53 default-demo-claim-sdd2t-pvc-d31fbcc9-8cab-4c28-b162-2349ff6350c3
-rw-r----- 1 root   root      52 Aug 23 16:56 .rmtab
```

```bash
ls -la /HDD5T/nfsshare/exports/SDD2T/default-demo-claim-sdd2t-pvc-d31fbcc9-8cab-4c28-b162-2349ff6350c3/
total 8
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:53 .
drwxrwxr-x 5 nobody nogroup 4096 Aug 23 16:53 ..
-rw-r--r-- 1 nobody nogroup    0 Aug 23 16:53 SUCCESS
```

Now delete the pods, the pv(s) and pvc(s) remain. If you delete the pvc, they are both deleted and the volume is archived

```bash
ls -la /HDD5T/nfsshare/exports/HDD5T
total 13
drwxrwxr-x 3 nobody nogroup 4096 Aug 23 17:09 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:54 archived-default-demo-claim-hdd5t-pvc-7ce33985-1dfe-4985-a015-a534f7e03bdd
-rw-r----- 1 root   root      52 Aug 23 17:09 .rmtab
```

```bash
ls -la /HDD5T/nfsshare/exports/SDD2T
total 13
drwxrwxr-x 5 nobody nogroup 4096 Aug 23 17:07 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:53 archived-default-demo-claim-sdd2t-pvc-d31fbcc9-8cab-4c28-b162-2349ff6350c3
-rw-r----- 1 root   root      52 Aug 23 17:09 .rmtab
```

If you create the pods again, new volumes will be created, allowing to get the contents back to the new volumes

```bash
ls -la /HDD5T/nfsshare/exports/HDD5T
total 17
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 17:10 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:54 archived-default-demo-claim-hdd5t-pvc-7ce33985-1dfe-4985-a015-a534f7e03bdd
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 17:10 default-demo-claim-hdd5t-pvc-cb501ec4-c418-42f8-856f-3b5b55571eff
-rw-r----- 1 root   root      52 Aug 23 17:14 .rmtab
```

```bash
ls -la /HDD5T/nfsshare/exports/SDD2T
total 17
drwxrwxr-x 6 nobody nogroup 4096 Aug 23 17:10 .
drwxrwxr-x 4 nobody nogroup 4096 Aug 23 00:16 ..
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 16:53 archived-default-demo-claim-sdd2t-pvc-d31fbcc9-8cab-4c28-b162-2349ff6350c3
drwxrwxrwx 2 nobody nogroup 4096 Aug 23 17:10 default-demo-claim-sdd2t-pvc-b973a974-145c-49c0-897e-b196b14d1c67
-rw-r----- 1 root   root      52 Aug 23 17:14 .rmtab
```

## Limitations and pitfalls

The software is subject to some limitations, listed below:

- The storage space provided is not guaranteed: you can allocate more than the total shared size via NFS, because there is no check for it.
- The provisioned storage limit is not enforced. The application can expand to use all the available storage regardless of the provisioned size.
- Storage resize/expansion operations are not presently supported in any form. You will end up in an error state: Ignoring the PVC: didn't find a plugin capable of expanding the volume; waiting for an external controller to process this PVC.
- The read-write permissions are not governed by the access-mode parameter, but by the settings used in the NFS configuration
