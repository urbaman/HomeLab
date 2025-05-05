# Install k8tz

To make all the pods use the same timezone:

```bash
helm repo add k8tz https://k8tz.github.io/k8tz/
helm repo update
helm upgrade -i k8tz -n kube-system k8tz/k8tz --set timezone=Europe/Rome
```

From now on, all of the pods created, will have the same selected timezone.
