# Redis cluster installation

Select the type of cluster:

- **Redis (master-slaves)**: Supports multiple databases. Single write point (single master)
- **Redis Cluster**: Supports only one database. Better if you have a big dataset. Multiple write points (multiple masters)

## Redis (master-slaves)

Install the chart.

```bash
helm upgrade -i redis oci://registry-1.docker.io/bitnamicharts/redis --namespace redis --create-namespace --set master.persistence.storageClass=rook-ceph-nvme2tb --set replica.persistence.storageClass=rook-ceph-nvme2tb --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true  --set metrics.serviceMonitor.additionalLabels.release=kube-prometheus-stack
```

From chart v19

```bash
helm upgrade -i redis oci://registry-1.docker.io/bitnamicharts/redis --namespace redis --create-namespace --set master.persistence.storageClass=rook-ceph-nvme2tb --set replica.persistence.storageClass=rook-ceph-nvme2tb --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true  --set metrics.serviceMonitor.additionalLabels.release=kube-prometheus-stack --set global.compatibility.openshift.adaptSecurityContext=disabled --set master.containerSecurityContext.runAsGroup=0 --set replica.containerSecurityContext.runAsGroup=0 --set metrics.containerSecurityContext.runAsGroup=0 --set master.containerSecurityContext.readOnlyRootFilesystem=false --set replica.containerSecurityContext.readOnlyRootFilesystem=false --set metrics.containerSecurityContext.readOnlyRootFilesystem=false --set master.resourcesPreset=none --set replica.resourcesPreset=none --set metrics.resourcesPreset=none
```

## Standalone

```bash
helm upgrade -i redis oci://registry-1.docker.io/bitnamicharts/redis --namespace redis --create-namespace --set architecture=standalone --set master.persistence.storageClass=rook-ceph-nvme2tb --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true  --set metrics.serviceMonitor.additionalLabels.release=kube-prometheus-stack
```

## Access

You can find the redis password in the `redis` secret:

```bash
kubectl get secret -n redis redis -o jsonpath='{.data.\redis-password}' | base64 -d
```

You can access through the `redis-master.redis.svc.cluster.local` service for read/writes.

The `redis-replica.redis.svc.cluster.local` connects to the replicas, in read-only access

## Expose through Traefik

Define an entrypoint for redis (port 6439) in Traefik, then deploy the `ig-redis.yaml` file


## Recover after a crash

If the cluster crashes, you'll probably have the following error:

```bash
Bad file format reading the append only file appendonly.aof.57.incr.aof: make a backup of your AOF file, then use ./redis-check-aof --fix <filename.manifest>
```

Create a pod with redis-client pointing to the persistent claim of the master

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: redis-client
  namespace: redis
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: <PERSISTENT-CLAIM>
  containers:
    - name: redis
      image: docker.io/bitnami/redis:6.2.3-debian-10-r0
      command: ["/bin/bash"]
      args: ["-c", "sleep infinity"]
      volumeMounts:
        - mountPath: "/tmp"
          name: data
EOF
```

And fix the problem.

```bash
kubectl exec -it -n redis redis-client -- redis-check-aof --fix /tmp/appendonlydir/appendonly.aof.57.incr.aof
```

Follow the instructions.
