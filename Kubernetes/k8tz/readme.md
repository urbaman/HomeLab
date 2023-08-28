# Install k8tz

To make all the pods use the same timezone:

```bash
helm upgrade --install k8tz k8tz/k8tz \
    --set timezone=Europe/Rome \
    --set replicaCount=3 \
    --set cronJobTimeZone=true
```

From now on, all of the pods created, will have the same selected timezone.
