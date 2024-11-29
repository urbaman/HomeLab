# Install prometheus pushgateway on kubernetes

Add the repo if not present, extract the values file

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm show values prometheus-community/prometheus-pushgateway > pushgateway-values.yaml
```

Define the namespace to monitoring, the Service Monitor to true and add the `release: kube-prometheus-stack` additional label to the Service Monitor.

```bash
helm upgrade -i prometheus-pushgateway prometheus-community/prometheus-pushgateway --values pushgateway-values.yaml -n monitoring
```

Then, create the ingressroute from the yaml template