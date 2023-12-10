# Install memcached

Install the helm chart, customizing the settings (we add high-availability, metrics and persistence)

```bash
helm upgrade -i memcached -n memcached --create-namespace oci://registry-1.docker.io/bitnamicharts/memcached --set architecture=high-availability --set autoscaling.enabled=true --set persistence.enabled=true --set persistence.storageClass=longhorn-nvme --set persistence.accessModes={ReadWriteMany} --set metrics.enabled=true --set metrics.serviceMonitor.enabled=true --set metrics.serviceMonitor.labels.release=kube-prometheus-stack --set metrics.resources.requests.cpu=250m --set metrics.resources.requests.memory=256Mi
```

Memcached can be accessed via port 11211 on the following DNS name from within your cluster: `memcached.memcached.svc.cluster.local`
