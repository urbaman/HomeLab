# Uptime Kuma for service uptime monitoring

## Install Uptime Kuma through helm

Once Helm has been set up correctly, add the repo as follows:

```bash
helm repo add uptime-kuma https://dirsigler.github.io/uptime-kuma-helm
helm repo update
helm show values uptime-kuma/uptime-kuma > uptime-kuma-values.yaml
```

If you had already added this repo earlier, run helm repo update to retrieve the latest versions of the packages. You can then run helm search repo uptime-kuma to see the charts.

Chaneg the values to install the latest image, on desired storageClass, with serviceMonitor enabled and with the proper serviceMonitor additionalLabels:

```bash
helm upgrade -i uptime-kuma uptime-kuma/uptime-kuma -n monitoring -f uptime-kuma-values.yaml
```

## Expose through IngressRoute

Create and install the ssl cert and ingressroute

```bash
kubectl apply -f ig-uptimekuma.yaml
```
