# Install k8tz

To make all the pods use the same timezone:

```bash
helm repo add k8tz https://k8tz.github.io/k8tz/
helm repo update
helm upgrade --install -n k8tz --create-namespace k8tz k8tz/k8tz \
    --set timezone=Europe/Rome \
    --set replicaCount=3 \
    --set cronJobTimeZone=true
```

From now on, all of the pods created, will have the same selected timezone.
