# Redis cluster installation

Select the type of cluster:

- **Redis (master-slaves)**: Supports multiple databases. Single write point (single master)
- **Redis Cluster**: Supports only one database. Better if you have a big dataset. Multiple write points (multiple masters)

## Redis (master-slaves)

Install the chart.

```bash
helm install redis oci://registry-1.docker.io/bitnamicharts/redis --namespace redis --create-namespace --set master.persistence.accessModes={ReadWriteMany} --set replica.persistence.accessModes={ReadWriteMany} --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true  --set metrics.serviceMonitor.additionalLabels.release=kube-prometheus-stack
```

You can access through the redis-master.redis.svc.cluster.local service for read/writes

The redis-replica.redis.svc.cluster.local connects to the replicas, in read-only access
